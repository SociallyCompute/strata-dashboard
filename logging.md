Right now there is regexing into event_data strings. Regex + $and on a blob won’t use indexes and will melt CPUs at scale.


Bad (double-encoded string):
```
"event_data": "{\"dialogueEventType\":\"DialogueNodeEvent\",\"conversationId\":\"95\",\"nodeId\":\"41\"}"
```
The problem with the above is that the event_data is a string, not an object.  This is why regex is having to be used. This is not indexable. 

Also, numerical values should not be quoted. IDs like conversationId/nodeId should be ints; positions (x,y,z) are floats.

Good (proper JSON object, indexable):
```
"event_data": {
  "dialogueEventType": "DialogueNodeEvent",
  "conversationId": 95,
  "nodeId": 41
}
```

Bad (double-encoded string and the use of "Key" and "Value"):
```
{
  "_id": "68db020b72ebf4f11406e6c0",
  "game": "mhs",
  "player_id": "test@example.com",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "event_type": "playerPositionEvent",
  "event_data": "{\"playerPosition\":{\"x\":-19.51862144470215,\"y\":105.34504699707031,\"z\":-65.71957397460938}}",
  "device_info": [
    {"Key": "platform", "Value": "WindowsEditor"},
    {"Key": "browser",  "Value": "WebGL_Browser"}
  ]
}
```
The device_info should be a subdocument, not an array. Maybe call it "device" instead of "device_info".

Better (proper JSON object, indexable):
```
{
  "_id": "68db020b72ebf4f11406e6c0",
  "game": "mhs",
  "player_id": "test@example.com",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "event_type": "playerPositionEvent",
  "event_data": {
    "playerPosition": {
      "x": -19.51862144470215,
      "y": 105.34504699707031,
      "z": -65.71957397460938
    }
  },
  "device": {
    "platform": "WindowsEditor",
    "browser": "WebGL_Browser"
  }
}
```

Instead of "event_data" call it "data".

```
{
  "_id": "68db020b72ebf4f11406e6c0",
  "game": "mhs",
  "player_id": "test@example.com",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "event_type": "playerPositionEvent",
  "data": { "playerPosition": { "x": -19.51, "y": 105.35, "z": -65.72 } },
  "device": { 
  	"platform": "WindowsEditor", 
  	"browser": "WebGL_Browser" 
  }
}
```

Mixing snake_case and camelCase in the same collection is confusing. It doesn’t change performance, but it does increase bugs, friction, and "wait which key was that?" moments. Pick one and use it everywhere. It also has an impact on translation of json to variable names in the server code; Go in our case (see below).

Which to choose?

For Mongo/DocumentDB + JSON payloads, I recommend camelCase:
- It’s idiomatic for JSON and the JS ecosystem (and Unity C# serializers map nicely).
- Most Mongo examples and tooling expect camelCase (createdAt, not created_at).
- Go structs can tag easily: json:"playerId" bson:"playerId".

Snake_case is fine too, but it’s more common in SQL schemas and Python code—not in JSON you pass around your web stack.

Canonical shape (camelCase)

Top-level = stable envelope, event-specific = data subdoc:

```
{
  "_id": "68db020b72ebf4f11406e6c0",
  "game": "mhs",
  "playerId": "test@example.com",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "eventType": "playerPositionEvent",
  "data": { "playerPosition": { "x": -19.51, "y": 105.35, "z": -65.72 } },
  "device": { "platform": "WindowsEditor", "browser": "WebGL_Browser" }
}
```

Translation of json into Go struct:

```
type LogEvent struct {
    Game       string            `json:"game"       bson:"game"`
    PlayerID   string            `json:"playerId"   bson:"playerId"`
    Timestamp  time.Time         `json:"timestamp"  bson:"timestamp"`
    EventType  string            `json:"eventType"  bson:"eventType"`
    Data       map[string]any    `json:"data"       bson:"data"`
    Device     map[string]string `json:"device"     bson:"device"`
    EventKey   string            `json:"eventKey,omitempty" bson:"eventKey,omitempty"`
}
```

Using snake_case would just be confusing in this context.

You will notice in the above struct "EventKey"/"eventKey".

"eventKey" is a composite, normalized key that rolls up the few fields you care about for ultra-fast matching/routing.

Think of it as a tiny fingerprint for the trigger, not a duplicate label.

When eventKey is useful:

Think of it as a tiny fingerprint for the trigger, not a duplicate label.
Composite trigger key:
- Dialogue node: eventKey = "dialogue:31:29" (from eventType=DialogueEvent, conversationId=31, nodeId=29)
- Quest finish: eventKey = "questFinish:21" (from questEventType=questFinishEvent, questId=21)
- Explicit progress event: eventKey = "pp:U2.C3"
Why bother?
- O(1) EOP checks: a single string compare instead of multiple field comparisons.
- Tiny single-field index: {"playerId":1,"eventKey":1,"timestamp":1} can replace several family-specific compound indexes if your hottest queries are exact matches on a few trigger signatures.
- Routing/partitioning: use eventKey as a topic-ish label for SNS/SQS routing, or as messageGroupId for FIFO ordering when needed.
- Cache map: in code, handlers[eventKey] is cleaner than a web of predicates.

When it’s not worth it
- If you already normalize data and have good compound indexes, and your code happily checks (eventType, data.conversationId, data.nodeId), you don’t need eventKey.
- If you set eventKey = eventType, that’s pure duplication (extra bytes, extra index width, zero benefit). An example where you wouldn't have an eventKey is for playerPosition since playerPosition:x:y:z is not likely to be a trigger for anything.

Practical recommendation for our context:
- Keep: eventType (canonical), timestamp (BSON Date), playerId, data (normalized, typed), device.
- Only add eventKey if you use it as a composite trigger key. Otherwise, omit it entirely.
- If you add it, standardize the format (short and unambiguous), e.g.:
- dialogue:<conversationId>:<nodeId>
- questFinish:<questId>
- pp:<pointId>

Example (with a meaningful key)
```
{
  "playerId": "alice@example.com",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "eventType": "DialogueEvent",
  "data": { "dialogueEventType":"DialogueNodeEvent", "conversationId":95, "nodeId":41 },
  "eventKey": "dialogue:95:41"   // composite, not redundant

```

Example (no key, perfectly fine)
{
  "playerId": "alice@example.com",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "eventType": "DialogueEvent",
  "data": { "dialogueEventType":"DialogueNodeEvent", "conversationId":95, "nodeId":41 }
}

Bottom line: either make eventKey carry extra signal (composite) or don’t store it at all. Keeping it equal to eventType just adds clutter with no upside.

When I say an event"acts as a trigger," I mean: the moment that event appears, it deterministically tells the system to do some work—compute a score, flip a progress point, award a badge, kick off a recompute, etc. Triggers are the opposite of passive telemetry (like position pings) that you just store.

Here’s how that plays out, concretely.

What counts as a trigger?
- Completion triggers (EOP): the last step of a progress point.
Example: DialogueEvent with conversationId=20 and nodeId=26 means U2.C2 is complete.
- Correction/attempt triggers: something that changes the eventual color (e.g., submitting an argument, requesting help).
- Teacher actions: reset/move-back events that require recomputing prior points.
- Milestones/achievements: e.g., questFinishEvent for quest 34 → award "Crash Survivor".

Non-triggers: high-volume signals you don’t act on immediately (e.g., playerPositionEvent), unless you explicitly define regions/checkpoints.

How a trigger is used (end-to-end example)

Progress point U2.C2 ("find Dr. Toppo < 5 min"):
1.	Event arrives to ingest
```
{
  "playerId": "alice@example.com",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "eventType": "DialogueEvent",
  "data": { "dialogueEventType": "DialogueNodeEvent", "conversationId": 20, "nodeId": 26 }
}
```

2.	Match against trigger rules (in-memory predicates or a tiny config(yaml)):
```
- when:
    eventType: DialogueEvent
    data.conversationId: 20
    data.nodeId: 26
  do:
    emit: pp:U2.C2
```
This match is the trigger.

3.	Publish a job (to SNS→SQS) for the materializer:
```
{
  "jobType": "ComputeProgressPoint",
  "pointId": "U2.C2",
  "playerId": "alice@example.com",
  "triggerEventId": "68db020c72ebf...",
  "ts": "2025-09-29T22:02:52.454Z"
}
```

(If you use FIFO SQS, set messageGroupId = playerId for per-player ordering.)

4.	Materializer computes
It queries the logs for the matching start/end timestamps, measures the duration, counts help events/attempts, maps to green/yellow, and upserts:
```
{ "playerId":"alice@example.com","pointId":"U2.C2","status":"green","score":4.0,"updatedAt":"..." }
```

5.	Dashboard stays cheap
The teacher’s view just reads progress_points. Missing row = white; stored row = green/yellow.

Where a composite eventKey helps

For triggers, you can precompute a compact key to speed checks/routing:
- Dialogue node: eventKey = "dialogue:20:26"
- Quest finish: eventKey = "questFinish:21"
- Explicit progress: eventKey = "pp:U2.C2"

Then your trigger table can be a simple set lookup (Go code):
```
if triggerSet[e.EventKey] { enqueueCompute("U2.C2", e.PlayerID, e.ID, e.Timestamp) }
```

If you don’t need this convenience (you’re happy checking the fields directly), skip eventKey.

Do position events ever "act as triggers"?

Only if you define them to—for example:
- Entering a named region: eventKey = "pos:region:temple" when (x,y,z) crosses a boundary.
- Hitting a checkpoint: eventKey = "pos:checkpoint:3".

If you don’t route on regions/checkpoints, playerPositionEvent is not a trigger—just store it.

Rule of thumb: a trigger is any event that, by itself, means "do work now." Use it to enqueue compute or side-effects immediately, so you never poll or re-query the DB to discover that "nothing happened yet."

The following is an alternative to the shape used in prior examples:
```
{
  "_id": "...",
  "game": "mhs",
  "playerId": "test@example.com",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "type": "DialogueEvent",
  "data": { "dialogueEventType": "DialogueNodeEvent", "conversationId": 95, "nodeId": 41 },
  "device": { "platform": "WindowsEditor", "browser": "WebGL_Browser" }
}
```

An index on the above is:
```
db.logs.createIndex({ playerId: 1, type: 1, "data.conversationId": 1, "data.nodeId": 1, timestamp: 1 })
```

An index is a sorted lookup table Mongo/DocumentDB keeps on the side so it can jump straight to the documents you want instead of scanning the whole collection. Think of it like the index in a book—organized by keys, pointing to page numbers (document locations).

### The example index
```
db.logs.createIndex({
  playerId: 1,
  type: 1,
  "data.conversationId": 1,
  "data.nodeId": 1,
  timestamp: 1
})
```

What this means:
- It’s a compound index (multiple fields, in this exact left-to-right order).
- 1 = ascending (use -1 for descending; matters for sorting).
- Internally it’s sorted by:
```
playerId → type → data.conversationId → data.nodeId → timestamp
```
- Each entry in the index stores those keys plus a pointer to the actual document.

### How Mongo/DocumentDB uses it

1) Leftmost-prefix rule

Queries can use any leftmost prefix of the index efficiently:
- Uses the index fully:
	- {playerId, type, data.conversationId, data.nodeId} (+ optional range on timestamp)
- Uses the prefix:
	- {playerId, type, data.conversationId}
	- {playerId, type}
	- {playerId}
- Won’t use it (or will use it poorly) if you start at the middle:
	- {type: "DialogueEvent", "data.conversationId": 31} (no playerId)

2) Equality → then range → then sort

- Put equality filters first in the index (here: playerId, type, data.conversationId, data.nodeId).

- Put the range/sort field after them (here: timestamp).

- That lets queries like this scream:

```
// exact filters, then sort/range on timestamp
db.logs.find({
  playerId, type: "DialogueEvent",
  "data.conversationId": 31, "data.nodeId": 29
}).sort({ timestamp: 1 })           // uses the index for sort
// or with a range:
db.logs.find({
  playerId, type: "DialogueEvent",
  "data.conversationId": 31, "data.nodeId": 29,
  timestamp: { $gte: ISODate("2025-09-01") }
})
```

3) Covered queries (extra fast)

If your projection only needs fields present in the index, Mongo can answer directly from the index without touching the full document ("covered query"). E.g., projecting just timestamp and the indexed keys.

4) Direction & sorting

If you want descending time scans:

```
db.logs.createIndex({ playerId: 1, type: 1, "data.conversationId": 1, "data.nodeId": 1, timestamp: -1 })
```

The sort direction must match the index to avoid an in-memory sort.

### Choosing good index order (rule of thumb)
1.	Equality fields first (most selective early if possible).
2.	Then the range or sort field (timestamp here).
3.	Keep types consistent (number vs string) so matches hit the index.

### What this index is perfect for in your app
-"Find the earliest node for player P in convo 31/node 29."
-"Fetch all node-29 hits for player P and sort by time."
-"Count attempts for player P in convo 70 where nodeId ∈ {7,25,33}."
(Use $in on data.nodeId; still uses the index.)

### What it’s not ideal for
- Queries without playerId (e.g., "show all players on node 29"). For those, you’d add a separate index starting with type/conversationId/nodeId.
- Position events with predicates on data.playerPosition.x/y/z—that likely needs a different index (or its own collection) if you ever query by ranges.

### Costs and cautions
- Write cost: every insert/update must update every index; don’t index everything—only what you query/sort on.
- Space: indexes take RAM/disk; keep them tight (avoid bloated fields, keep numbers as numbers).
- Explain it: use .explain("executionStats") to verify the planner picks your index and to see scanned keys vs returned docs.

### Tiny Go snippet (creating index)
```
keys := bson.D{
  {Key: "playerId", Value: 1},
  {Key: "type", Value: 1},
  {Key: "data.conversationId", Value: 1},
  {Key: "data.nodeId", Value: 1},
  {Key: "timestamp", Value: 1},
}
model := mongo.IndexModel{Keys: keys}
_, _ = coll.Indexes().CreateOne(ctx, model)
```

To optimize quest logic, you might also add:
```
db.logs.createIndex({ playerId: 1, type: 1, "data.questId": 1, "data.questEventType": 1, timestamp: 1 })
```

That’s the gist: pick indexes that mirror your most common filters + sorts, put equality fields first, and keep data types consistent so the index actually matches.

## Implementing the Dashboard

Blueprint to implement
1.	Log write (ingest service):
- Persist event to DocumentDB.
- If eventType in EOP_SET, publish ComputeProgressPoint to SQS: {playerId, pointId, triggerEventId, ts}.
2.	Worker (materializer):
- Consume messages (max-batch N).
- For each (playerId, pointId):
- Run the Mongo aggregation to compute score.
- Map to status via thresholds (green/yellow).
- findOneAndUpdate(..., upsert:true) into progress_points with score_version.
- On error → retry; on repeated error → DLQ.
3.	Dashboard API:
- Given group_id, fetch playerIds then bulk fetch progress_points.
- Return a dense matrix the UI can paint in one go.
4.	Freshness loop:
- HTMX poll every 2–5s (or SSE) for "changed since T" to update individual cells without repainting the page.

### What is SQS?

Amazon Simple Queue Service (SQS) is AWS’s fully managed message queue. Producers drop messages into a queue; consumers poll and process them asynchronously. It’s built to decouple services and smooth out spikes so your ingest path doesn’t explode when downstream gets busy.

Key bits:
- Queue types:
- Standard: massive throughput, at-least-once delivery, best-effort ordering.
- FIFO: lower throughput, ordered per message group, and deduplication (aimed at "exactly-once" processing when your handler is idempotent).
- Operational goodies: visibility timeouts, dead-letter queues (DLQ), long polling, server-side encryption, message retention, per-message delay.

What’s an "SQS fan-out""?

Colloquially, folks say "SQS fan-out," but the classic fan-out pattern is SNS (pub/sub) → SQS (queues):

```
         (one event)
Producer  ──►  SNS Topic  ──►  SQS Queue A  ──► Consumer A
                         ╰──►  SQS Queue B  ──► Consumer B
                         ╰──►  SQS Queue C  ──► Consumer C
```

You publish once to an SNS topic (or an EventBridge bus).
- Multiple SQS queues subscribe and each receives its own copy of the message.
- Each downstream consumer scales, retries, and fails independently (their own concurrency, visibility timeout, and DLQ).
- If you need ordered fan-out, use SNS FIFO → SQS FIFO with MessageGroupId (e.g., the playerId) to preserve per-player order.

Why this rocks:
- Loose coupling: producers don’t know who’s listening.
- Easy expansion: add a new consumer later—just attach another queue.
- Back-pressure isolation: analytics can lag without slowing your ingest or your materializer.

Variants & gotchas
- You can"fan-out" by pushing the same message to multiple SQS queues directly from the producer, but that hard-codes destinations and doubles your failure modes. SNS/EventBridge is cleaner.
- With Standard queues, design for idempotency (retries/duplicates can happen). FIFO helps when you truly need order/dedup, at some throughput cost.
- Always wire a DLQ and alarms (queue depth, age, DLQ count).

In our context

For the progress-point pipeline:
- Logging server publishes an EOP event to SNS.
- Subscribe separate SQS queues for:
- Progress Materializer (compute green/yellow + upsert view),
- Achievements/Badges,
- Analytics/Audit.
- Each consumer can scale on its own schedule. Your ingest stays thin and fast; your dashboard reads a precomputed view.

### Make EOP detection a single-event match (no queries)

Pick, for each progress point, a canonical finishing event that appears in the log stream. Example candidates:
- U1.C1 → DialogueEvent with conversationId=31 and nodeId=29
- U1.C2 → DialogueEvent with conversationId=30 and nodeId=98
- U2.C1 → questEvent with questFinishEvent and questID=21
- U2.C4 → DialogueEvent with conversationId=23 and nodeId=17
- U2.C7 → DialogueEvent with conversationId=20 and nodeId in {74,75} (then materializer checks the rest)

For any progress point where you can’t name a unique last step (e.g., "whichever of two conversations finishes last"), pick the latest of the known steps as the EOP; the materializer can still look back to compute green/yellow.

**Key idea: EOP detection should never require a DB read; it should be "does this single incoming event match an EOP signature?". That keeps ingest constant-time.**

### Put EOP signatures in a tiny "trigger catalog"

Make it declarative so game/analytics folks can adjust without code rewrites (yaml):

```
# progress_triggers.yaml
- pointId: U1.C1
  match:
    eventType: DialogueEvent
    data:
      dialogueEventType: DialogueNodeEvent
      conversationId: 31
      nodeId: 29

- pointId: U2.C1
  match:
    eventType: questEvent
    data:
      questEventType: questFinishEvent
      questID: 21

- pointId U2.C7
  match_any:
    - { eventType: DialogueEvent, data: { conversationId: 20, nodeId: 74 } }
    - { eventType: DialogueEvent, data: { conversationId: 20, nodeId: 75 } }
```
At startup, load and compile this into in-memory predicates.

### Where to run detection (two good options)

A) Change Stream watcher (preferred):

A small Go service subscribes to the logs collection insert stream (Mongo/DocumentDB Change Streams). Note: ensure change streams are enabled on the cluster/parameter group; you only need to watch inserts. For each insert, evaluate the trigger predicates; on match, publish ComputeProgressPoint(playerId, pointId, triggerEventId, ts) to SNS→SQS. Logging server stays skinny.

B) On the logging server write-path (also fine):

After you persist the doc, run the same in-memory predicate check. If match ⇒ publish to SNS. No DB reads, just the event you’re already holding.

Either way, this is O(1) and microseconds per event.


The materializer’s job: on each queued EOP, run the complex aggregation for (playerId, pointId), compute score, map to {green|yellow}, and upsert into a progress_points collection. Missing row = white. Recomputes are idempotent (use triggerEventId and updatedAt).

### What about the tricky progress points?

- Durations / attempts (U2.C2/C3/C5/C7): EOP is a single finishing event (e.g.,"found Dr. Toppo" or "argument submitted"). The materializer then looks back to find the corresponding "start" event(s) and counts. That’s one heavy query per completion, not per dashboard refresh.
- Teacher rewind/reset: Treat the reset as its own "EOP" for all affected points (or a "Reset" control event). The watcher publishes compute jobs for those points; the materializer recomputes (often resulting in deletion or setting them to white/earlier state). Keep it idempotent.
- Out-of-order logs: No problem, compute replays history. If a "start" arrives late, the next meaningful event will re-trigger an EOP and recompute. With SQS Standard, ensure idempotent upsert; if you truly need per-player ordering, use SQS FIFO with MessageGroupId=playerId.

### Why this solves the "white" problem

You never query "hasn’t happened yet." The dashboard paints:
- white if there is no row in progress_points for (player, point).
- green/yellow from the stored row.
Only EOP transitions cause work, so 1,000 players doesn’t mean 1,000 repeated queries—just 1,000 once, when they actually finish.

### If we can change the game client, do this one small thing

Emit a single explicit event when a progress point completes, e.g.,
{ eventType: "ProgressPointCompleted", progressPointId: "U2.C3" }.
This shrinks EOP detection to a trivial exact match and eliminates any ambiguity about "which dialogue node is the last one." It’s the highest-ROI instrumentation we can add.

### Progress view model & indexes (the read side)

Row shape for progress_points

```
{
  "playerId": "alice@example.com",
  "pointId": "U2.C2",
  "status": "green",                 // "yellow" | "green"
  "score": 4.0,                      // optional
  "scoreVersion": 3,                 // bump on logic change
  "updatedAt": "2025-09-29T22:03:03Z",
  "lastTriggerId": "68db020c72ebf..."// idempotency token
}
```

- Indexes (add the read patterns):

```
db.progress_points.createIndex({ playerId: 1, pointId: 1 }, { unique: true, name: "uniq_player_point" })
db.progress_points.createIndex({ playerId: 1, updatedAt: -1 })   // fast refresh per player/group
// Only if you do cohort reports:
db.progress_points.createIndex({ pointId: 1, status: 1 })
```

Caching: for large groups, a 5–15s Redis cache of the rendered grid (keyed by groupId) keeps dashboards snappy.

Notes:

Normalization rules: "Always lowercase emails," "never stringify data," "timestamps are BSON Date."

Multitenancy keys: if hosting multiple orgs/games, include orgId/game in indexes and docs.


#### Security & privacy

PII handling: you’re using emails as playerId. Decide whether to store a hashed surrogate alongside (playerKey = sha256(lower(email))) for cross-system joins without exposing raw emails.

PII handling = how you limit, protect, and responsibly use personally identifiable information (names, emails, student IDs, etc.). In your pipeline the only hard PII you’re carrying is the email used as playerId. Good practice is to minimize PII spread: keep it in one place, and use a pseudonymous key everywhere else.

The idea: a hashed surrogate key

Instead of putting raw emails into every log row, derive a stable, non-reversible key from the email and use that in logs, metrics, exports, and joins.

Do this, not plain SHA-256:
Use a keyed HMAC (HMAC-SHA256) with a secret “pepper.” Emails are low-entropy; a plain hash can be reversed with a dictionary. HMAC prevents that.
- Input: canonicalEmail = lower(trim(email)) (also normalize Unicode if needed)
- Secret: pepper (random 256-bit value stored in KMS/SSM)
- Output: playerKey = HMAC_SHA256(pepper, canonicalEmail) (hex or base64)
- Optional: include orgId in the input if you want per-org unlinkability: HMAC(pepper, orgId || ":" || canonicalEmail)

Where it lives & how it’s used

1) Mapping you keep (private)

A small table the auth/user service owns:

```
// users collection (private)
{
  "userId": "u_2f3…",                 // internal stable id
  "playerId": "alice@example.com",     // email (PII)
  "playerKey": "hmac256:V1:…",         // pseudonymous key
  "orgId": "school-42",
  "keyVersion": 1,                     // supports rotation
  "createdAt": "...", "updatedAt": "..."
}
```

Indexes:
```
db.users.createIndex({ playerId: 1 }, { unique: true })
db.users.createIndex({ playerKey: 1 }, { unique: true })
```

2) What logs store (operational data)

Use playerKey, not the email, in logs and progress_points:

```
// logs
{
  "playerKey": "hmac256:V1:…",
  "orgId": "school-42",
  "timestamp": "2025-09-29T22:02:51.04Z",
  "type": "DialogueEvent",
  "data": { "dialogueEventType": "DialogueNodeEvent", "conversationId": 95, "nodeId": 41 },
  "device": { "platform": "WindowsEditor", "browser": "WebGL_Browser" }
}
// progress_points
{
  "playerKey": "hmac256:V1:…",
  "pointId": "U2.C2",
  "status": "green",
  "score": 4.0,
  "scoreVersion": 3,
  "updatedAt": "..."
}
```

Indexes update accordingly:

```
db.logs.createIndex({ playerKey: 1, type: 1, "data.conversationId": 1, "data.nodeId": 1, timestamp: 1 })
db.progress_points.createIndex({ playerKey: 1, pointId: 1 }, { unique: true })
```

3) How the app queries

- Ingest receives an email → compute playerKey at the edge → write playerKey into logs.
- Teacher dashboard works with players (emails). The API layer maps emails → playerKeys via the private users table, then queries logs/progress_points by playerKey. The UI never needs to see playerKey; it’s just the join key.

4) Sharing data safely

When you export to analytics or hand results to researchers, include playerKey (and maybe orgId), not emails. Teams can still join datasets on playerKey without seeing PII.

Why it’s better

- Minimizes PII sprawl: raw emails exist only in the users service.
- Safer if leaked: HMAC’d keys are not practically reversible.
- Consistent joins: same email → same playerKey → easy cross-dataset joins.
- Configurable privacy: add orgId into the HMAC input to prevent cross-org linking if that matters for FERPA/privacy.

Implementation notes

Go example (HMAC-SHA256)

```
import (
  "crypto/hmac"
  "crypto/sha256"
  "encoding/hex"
  "strings"
)

func PlayerKey(pepper []byte, email, orgId string) string {
  canon := strings.ToLower(strings.TrimSpace(email))
  mac := hmac.New(sha256.New, pepper)
  mac.Write([]byte(orgId))
  mac.Write([]byte(":"))
  mac.Write([]byte(canon))
  sum := mac.Sum(nil)
  return "hmac256:V1:" + hex.EncodeToString(sum) // include version prefix
}
```

**There is an assumption that email addresses (ids) do not change.**

- Store pepper in KMS/SSM; load at startup; never log it.
- Prefix with hmac256:V1: so you can rotate later (V2) and keep both during a transition.

Key rotation (probably not in our case)

- Add keyVersion to the users table and playerKey values (hmac256:V1:…).
- When rotating pepper, compute both V1 and V2 for a period; new logs use V2; the materializer reads either; optionally backfill if desired.

Pseudonymous ≠ anonymous

This is pseudonymization. Combined with timestamps/contexts, a motivated attacker may still re-identify. Keep access controls tight, log access to the users table, and avoid mixing unnecessary attributes in exports.

Bottom line: compute playerKey = HMAC_SHA256(pepper, lower(email)) (optionally with orgId), store playerKey everywhere, keep raw emails only in the users service, and update  indexes and joins to use playerKey. This gives linkability across systems without spraying PII around the data lake.




