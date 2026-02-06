# Need additional data
1. Would like greater density of in game position data. It starts when you hit the movement keyboard, or after a long movement it may capture another position log but the density is not enough to capture the exploration area.  
    - When a person keeps pressing WASD we don't get changing position data
    - Mouse interaction activities also seem incomplete. We do get when the players interact with the dialogue system. 
    - When players open in game tools we do get the open and close activities. 
    - Node interactions in the argumentation system are captured
2. **Logs that capture players interacting with in game objects are not getting created this time**, and that data is rich. We are only capturing the dialogue activity in the current version, but not interactions with the objects. (We do capture one object in unit 4, in the dungeon, we get logs for interactions with water tube type for solving the puzzle.)
3. For in game tool usage we would like more messages than the "open" and "close" event, such as how the in game tool is used, etc. Right now it seems like in the current version only the argumentation usage is logged. The **general** in game tool usage is not logged yet. There are currently 3 in game tools we would like tool use and how the user interacts specifically with the tool (these are ones we thought would be in this version, fwiw): 
    1. Map
    2. Chat Log
    3. Quest Panel (Which quest they have completed, which one they are focusing on right now, things like that.)
4. We also understand there will be "mini games" in a future version, and rich logs from those "mini games" will also be helpful

# Insights we are missing without this data: 
1. We are looking for clearer signals of guessing behavior and 
2. Off task behavior
3. Frustrated efforts / "giving up" ... in this version of the game the teachers may be better able to notice on site. The students want to learn something from the tasks, but they cannot figure out the game play part of it, so they get frustrated. 

**Question** Will we have access to the post game survey data that the consulting firm will be doing? This may aid us with early triangulation of those outcomes and in game behavior. 

**Idea**: 
- Since the dashboard has a design goal of providing specific intervention advice for the students, we would like to propose using a GenAI / MCS Server to generate some of this advice. We'd like to create agent so the logging system can provide specific intervention text. We think we can use learning theories and MHS instructor guide information to provide advice to students and teachers that are users. We perform some prompt engineering and give output to the narrative suggestions on the dashboard. 
    - We'd like to pilot/explore this starting soon
    - Then see what we can usefully what can be generated
    - Wenyi is thinking about how to apply AI Agent technology, and we'd like to think about how to explore this possibility. 
    - We don't think it will require the game engine to deploy. We think Dale, Wenyi, and Sean can design and implement these ideas into the dashboard. 

