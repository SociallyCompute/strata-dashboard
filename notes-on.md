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
3. Frustrated efforts / "giving up" ... in this version of the game the teachers may be better able to notice on site. 

