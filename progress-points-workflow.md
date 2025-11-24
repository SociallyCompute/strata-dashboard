<table style="border-collapse: collapse; width: 100%;">
  <tr>
    <th style="border: 1px solid #ccc; padding: 6px;">Progress Point Key</th>
    <th style="border: 1px solid #ccc; padding: 6px;">Progress End Event (Phase 1)</th>
    <th style="border: 1px solid #ccc; padding: 6px;">Script For Colors (Phase 2)</th>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U1.C1</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>has_event = coll.count_documents({
    "version": "Version [3.59] - 11/14/2025",
    "playerId": pid,
    "eventKey": "DialogueNodeEvent:31:29"
}) &gt; 0</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>if has_event:
    color = 1 ("green")
else:
    color = 0 ("white")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U1.C2</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>has_event = coll.count_documents({
    "version": "Version [3.59] - 11/14/2025",
    "playerId": pid,
    "eventKey": "DialogueNodeEvent:30:98"
}) &gt; 0</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>if has_event:
    color = 1 ("green")
else:
    color = 0 ("white")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U1.C3</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
  <pre><code>
  finishQueue = []
  for pid in allPlayers:
          has_event = coll.count_documents({
          "version": "Version [3.59] - 11/14/2025",
          "playerId": pid,
          "eventKey": "QuestActiveEvent:34"
          }) &gt; 0
     if has_event:
        finishQueue.append(pid)
     else:
        color = 0  # ("white")
</code></pre>
  </td>
  <td style="border: 1px solid #ccc; padding: 6px;">
    <pre><code>for pid in finishQueue:

    has_yellow_trigger = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": {"$in": [
            "DialogueNodeEvent:70:25",
            "DialogueNodeEvent:70:33"
        ]}
    }) &gt; 0

    if has_yellow_trigger:
        color = 2  # ("yellow")
    else:
        color = 1  # ("green")
</code></pre>
  </td>
</tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U1.C4</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>has_event = coll.count_documents({
    "version": "Version [3.59] - 11/14/2025",
    "playerId": pid,
    "eventKey": "QuestFinishEvent:34"
}) &gt; 0</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>if has_event:
    color = 1 ("green")
else:
    color = 0 ("white")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U2.C1</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>finishQueue = []
    For pid in allPlayers:
    has_event = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": "QuestFinishEvent:21"
    }) &gt; 0
    if has_event:
        finishQueue.append(pid)
    else:
        color = 0 #("white")</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>yellow_nodes = ["DialogueNodeEvent:68:23", "DialogueNodeEvent:68:27", "DialogueNodeEvent:68:28", "DialogueNodeEvent:68:31"]
success_node = "DialogueNodeEvent:68:29"
for pid in finishQueue:
    has_29 = coll.find_one({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid, 
        "eventKey": success_node
    }) is not None
    has_any_yellow = coll.find_one({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid, 
        "eventKey": {"$in": yellow_nodes}
    }) is not None
    if has_29 and not has_any_yellow:
        color = 1 #("green")
    else:
        color = 2 #("yellow")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U2.C2</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>finishQueue = []
For pid in allPlayers:
    has_event = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": "DialogueNodeEvent:20:26"
    }) &gt; 0
    if has_event:
        finishQueue.append(pid)
    else:
        color = 0 #("white")</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>START_KEY = "DialogueNodeEvent:20:1"
END_KEY = "DialogueNodeEvent:19:46"
TARGET_KEYS = [
    "DialogueNodeEvent:18:99",  "DialogueNodeEvent:28:179", "DialogueNodeEvent:59:179",
    "DialogueNodeEvent:18:223", "DialogueNodeEvent:28:182", "DialogueNodeEvent:59:182",
    "DialogueNodeEvent:18:224", "DialogueNodeEvent:28:183", "DialogueNodeEvent:59:183"
]
for pid in finishQueue:
    start_doc = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventType": "DialogueNodeEvent",
            "eventKey": START_KEY
        },
        sort=[("timestamp", 1)]
    )
    start_ts = datetime.fromisoformat(start_doc["timestamp"].replace("Z", "+00:00"))
    end_doc = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventType": "DialogueNodeEvent",
            "eventKey": END_KEY,
            "timestamp": {"$gte": start_doc["timestamp"]}
        },
        sort=[("timestamp", 1)]
    )
    end_ts = datetime.fromisoformat(end_doc["timestamp"].replace("Z", "+00:00"))
    start_iso = start_ts.isoformat().replace("+00:00", "Z")
    end_iso   = end_ts.isoformat().replace("+00:00", "Z")
    count_targets = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": {"$in": TARGET_KEYS},
        "timestamp": {
            "$gte": start_iso,
            "$lte": end_iso
        }
    })

    if count_targets &lt;= 1:
        color = 1 #("green")
    else:
        color = 2 #("yellow")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U2.C3</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>finishQueue = []

For pid in allPlayers:

    has_event = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": "DialogueNodeEvent:22:18"
    }) &gt; 0

    if has_event:
        finishQueue.append(pid)
    else:
        color = 0 #("white")</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>START_KEY = "DialogueNodeEvent:20:33"
END_KEY   = "DialogueNodeEvent:22:1"

TARGET_KEYS = [
    "DialogueNodeEvent:18:225", "DialogueNodeEvent:28:185", "DialogueNodeEvent:59:185",
    "DialogueNodeEvent:28:184", "DialogueNodeEvent:28:191", "DialogueNodeEvent:59:184", "DialogueNodeEvent:59:191",
    "DialogueNodeEvent:18:226", "DialogueNodeEvent:18:227", "DialogueNodeEvent:28:186", "DialogueNodeEvent:59:186",
    "DialogueNodeEvent:18:228", "DialogueNodeEvent:28:187", "DialogueNodeEvent:59:187",
    "DialogueNodeEvent:18:229", "DialogueNodeEvent:28:188", "DialogueNodeEvent:59:188",
    "DialogueNodeEvent:18:230", "DialogueNodeEvent:28:180", "DialogueNodeEvent:59:180",
    "DialogueNodeEvent:18:233", "DialogueNodeEvent:28:192", "DialogueNodeEvent:59:192",
    "DialogueNodeEvent:18:234", "DialogueNodeEvent:28:193", "DialogueNodeEvent:59:193",
    "DialogueNodeEvent:18:235", "DialogueNodeEvent:28:194", "DialogueNodeEvent:59:194",
    "DialogueNodeEvent:18:236", "DialogueNodeEvent:18:237", "DialogueNodeEvent:28:190", "DialogueNodeEvent:59:190"
]

for pid in finishQueue:

    start_doc = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventType": "DialogueNodeEvent",
            "eventKey": START_KEY
        },
        sort=[("timestamp", 1)]
    )

    start_ts = datetime.fromisoformat(start_doc["timestamp"].replace("Z", "+00:00"))

    end_doc = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventType": "DialogueNodeEvent",
            "eventKey": END_KEY,
            "timestamp": {"$gte": start_doc["timestamp"]}
        },
        sort=[("timestamp", 1)]
    )

    end_ts = datetime.fromisoformat(end_doc["timestamp"].replace("Z", "+00:00"))

    start_iso = start_ts.isoformat().replace("+00:00", "Z")
    end_iso   = end_ts.isoformat().replace("+00:00", "Z")

    count_targets = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": {"$in": TARGET_KEYS},
        "timestamp": {
            "$gte": start_iso,
            "$lte": end_iso
        }
    })

    if count_targets &lt;= 6:
        color = 1 #("green")
    else:
        color = 2 #("yellow")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U2.C4</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>finishQueue = []

For pid in allPlayers:

    has_event = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": "DialogueNodeEvent:23:17"
    }) &gt; 0

    if has_event:
        finishQueue.append(pid)
    else:
        color = 0 #("white")</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>success_key = "DialogueNodeEvent:74:21"

bad_keys = [
    "DialogueNodeEvent:74:16",
    "DialogueNodeEvent:74:17",
    "DialogueNodeEvent:74:20",
    "DialogueNodeEvent:74:22"
]

For pid in finishQueue:

    has_74_21 = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventKey": success_key
        }
    ) is not None

    has_bad_feedback = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventKey": {"$in": bad_keys}
        }
    ) is not None

    if has_74_21 and not has_bad_feedback:
        color = 1 #("green")
    else:
        color = 2 #("yellow")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U2.C5</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>finishQueue = []

For pid in allPlayers:

    has_event = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": "DialogueNodeEvent:23:42"
    }) &gt; 0

    if has_event:
         finishQueue.append(pid)
    else:
        color = 0 #("white")</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>POS_KEYS = [
    "DialogueNodeEvent:23:140", "DialogueNodeEvent:23:142", "DialogueNodeEvent:23:143",
    "DialogueNodeEvent:23:146", "DialogueNodeEvent:23:147", "DialogueNodeEvent:23:148",
    "DialogueNodeEvent:23:165", "DialogueNodeEvent:23:166", "DialogueNodeEvent:23:167",
    "DialogueNodeEvent:23:168", "DialogueNodeEvent:23:169", "DialogueNodeEvent:23:170",
    "DialogueNodeEvent:23:172", "DialogueNodeEvent:23:173", "DialogueNodeEvent:23:174",
    "DialogueNodeEvent:23:175", "DialogueNodeEvent:23:177", "DialogueNodeEvent:23:178",
    "DialogueNodeEvent:23:179", "DialogueNodeEvent:23:180", "DialogueNodeEvent:23:181",
    "DialogueNodeEvent:23:182", "DialogueNodeEvent:23:183", "DialogueNodeEvent:23:184",
    "DialogueNodeEvent:23:185", "DialogueNodeEvent:23:186"
]

NEG_KEYS = [
    "DialogueNodeEvent:26:137", "DialogueNodeEvent:26:144", "DialogueNodeEvent:26:145",
    "DialogueNodeEvent:26:187", "DialogueNodeEvent:26:188", "DialogueNodeEvent:26:189",
    "DialogueNodeEvent:26:191", "DialogueNodeEvent:26:192", "DialogueNodeEvent:26:193",
    "DialogueNodeEvent:26:194", "DialogueNodeEvent:26:195", "DialogueNodeEvent:26:196",
    "DialogueNodeEvent:26:197", "DialogueNodeEvent:26:198", "DialogueNodeEvent:26:199",
    "DialogueNodeEvent:26:200", "DialogueNodeEvent:26:201", "DialogueNodeEvent:26:202",
    "DialogueNodeEvent:26:203", "DialogueNodeEvent:26:204", "DialogueNodeEvent:26:205",
    "DialogueNodeEvent:26:206", "DialogueNodeEvent:26:207", "DialogueNodeEvent:26:208",
    "DialogueNodeEvent:26:209", "DialogueNodeEvent:26:210", "DialogueNodeEvent:26:211"
]

For pid in finishQueue:

    pos_count = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventType": "DialogueNodeEvent",
        "eventKey": {"$in": POS_KEYS}
    })

    neg_count = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventType": "DialogueNodeEvent",
        "eventKey": {"$in": NEG_KEYS}
    })

    score = pos_count - (neg_count / 3.0)

    if score &gt;= 4:
        color = 1 #("green")
    else:
        color = 2 #("yellow")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U2.C6</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>finishQueue = []

For pid in allPlayers:

    has_event = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": "DialogueNodeEvent:18:284"
    }) &gt; 0

    if has_event:
        finishQueue.append(pid)
    else:
        color = 0 #("white")</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>For pid in finishQueue:

    pass_node = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventKey": pass_node
        }
    )

    if not pass_node:
        color = 2 ("yellow")
        continue

    yellow_44_45 = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventKey": {"$in": yellow_node}
        }
    )

    if yellow_44_45:
        color = 2 ("yellow")
    else:
        color = 1 ("green")</code></pre>
    </td>
  </tr>

  <tr>
    <td style="border: 1px solid #ccc; padding: 6px;">U2.C7</td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>finishQueue = []

For pid in allPlayers:

    has_event = coll.count_documents({
        "version": "Version [3.59] - 11/14/2025",
        "playerId": pid,
        "eventKey": {"$in": ["DialogueNodeEvent:20:74", "DialogueNodeEvent:20:75"]}
    }) &gt; 0

    if has_event:
        finishQueue.append(pid)
    else:
        color = 0 ("white")</code></pre>
    </td>
    <td style="border: 1px solid #ccc; padding: 6px;">
      <pre><code>SUCCESS_KEY = "DialogueNodeEvent:27:7"

NEG_KEYS = [
    "DialogueNodeEvent:27:11", "DialogueNodeEvent:27:12", "DialogueNodeEvent:27:13", "DialogueNodeEvent:27:14",
    "DialogueNodeEvent:27:15", "DialogueNodeEvent:27:16", "DialogueNodeEvent:27:17", "DialogueNodeEvent:27:18",
    "DialogueNodeEvent:27:19", "DialogueNodeEvent:27:20", "DialogueNodeEvent:27:21", "DialogueNodeEvent:27:22",
    "DialogueNodeEvent:27:23", "DialogueNodeEvent:27:24", "DialogueNodeEvent:27:25", "DialogueNodeEvent:27:26",
    "DialogueNodeEvent:27:27", "DialogueNodeEvent:27:28", "DialogueNodeEvent:27:29", "DialogueNodeEvent:27:30"
]

For pid in finishQueue:

    has_success = coll.find_one(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventKey": SUCCESS_KEY
        }
    ) is not None

    neg_count = coll.count_documents(
        {
            "version": "Version [3.59] - 11/14/2025",
            "playerId": pid,
            "eventKey": {"$in": NEG_KEYS}
        }
    )

    if has_success and neg_count &lt;= 3:
        color = 1 ("green")
    else:
        color = 2 ("yellow")</code></pre>
    </td>
  </tr>
</table>

