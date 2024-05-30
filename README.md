# This or That Application

<p align="center"><img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-1-2.png" width="600" alt="GetPlayers Diagram"/></p>
<p align="center"><a href="https://choose-this-or-that-3f8c620e977f.herokuapp.com/">Demo Link</a></p>

## Table of Content
1. [What exactly is this app doing?](#section1)  
2. [Core functionalities supported](#section2)
    -  2.1. [Users can create a room and list the items or people they want others to vote on](#section2.1)
3. [What exactly is this `rating score`?](#section3)
4. [APIs developed](#section4)
5. [Database schema](#section5)
6. [How do we get all players combinations to be paired?](#section6)
7. [Why do I use MongoDB compared to other database option?](#section7)
8. [Future works](#section8)
9. [Helpful links that helped me during the project](#section9)

<a name="section1"/>

## 1. What exactly is this app doing?
This app enables users to create multiple comparison rooms where votes determine the rankings. Users can set up a room, list items or people for comparison, and invite others to vote. It is inspired by Mark Zuckerberg's [Facemash application](https://thesocialnetwork.fandom.com/wiki/Facemash).

<a name="section2"/>

## 2. Core functionalities supported

<a name="section2.1"/>

### 2.1. Users can create a room and list the items or people they want others to vote on.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/createNewRoomFlow.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-7.png" width="600" alt="GetPlayers Diagram"/>
</p>

### 2.2. Users can edit a room, insert new items or people (let's call it `Object of Comparison`), or delete existing items or people.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/editRoomFlow3.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-4.png" width="600" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-5-2.png" width="600" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-6-2.png" width="600" alt="GetPlayers Diagram"/>
</p>

### 2.3. Users can get a room, vote, and have the `rating score` updated on the backend.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/playFlow.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-2-2.png" width="600" alt="GetPlayers Diagram"/>
</p>

### 2.4. Users can see the leaderboard of that particular room, which contains a list of active `Object of Comparison` sorted in descending order of the `rating score`, as well as showing the total number of people voting on the room.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/leaderboardFlow.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-3-2.png" width="600" alt="GetPlayers Diagram"/>
</p>

### 2.5. Users can see the overall statistics of the site, seeing how many users voted across all the rooms, and the most popular `roomId` by the total number of users voted on that room.
<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/seeStatisticsFlow.png" alt="GetPlayers Diagram"/>
  <br/><br/>
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/this-or-that-pic-1-2.png" width="600" alt="GetPlayers Diagram"/>
</p>

<a name="section3"/>

## 3. What exactly is this `rating score`?
This `rating score` is [Elo-rating](https://en.wikipedia.org/wiki/Elo_rating_system#Theory). Let's say two `Object of Comparison` compete, Object A and Object B. Object A has a rating of 1500 ($R_A = 1500$), while Object B has a rating of 1000 ($R_B = 1000$). We can calculate the expected score of Object A (expressed as $E_A$) and the expected score of Object B ($E_B$) using below's formula:

$$ E_A = {1 \over 1 + 10 ^ {(R_B - R_A)/400}} $$

$$ E_B = {1 \over 1 + 10 ^ {(R_A - R_B)/400}} $$

Inserting $R_A = 1500$, $R_B = 1000$, we got $E_A = 0.948$ and $E_B = 0.052$ (Notice that $E_A + E_B = 1$). This means that, if the winning score is 1, Object A is expected to score 0.948 points, while Object B is expected to score only 0.052.

To calculate the new score of Object A ($R_A'$) and the new score of ($R_B'$), we can use this formula:

$$R_A' = R_A + K(S_A - E_A)$$ 

$$R_B' = R_B + K(S_B - E_B)$$

Where $S_A$ and $S_B$ are the scores of A and B respectively. If Object A wins, then $S_A = 1$ and $S_B = 0$. Likewise, if Object 2 wins, then $S_A = 0$ and $S_B = 1$. The adjustment factor, the $K$, is set to 32.

If Object A wins the game against Object B, plugging in the value for new rating, we got $R_A' = 1502$ and $R_B' = 998$ (difference in rating $\pm 2$). On the contrary, if Object B pulls a surprise win against Object A, we get $R_A' = 1470$ and $R_B' = 1030$ (difference in rating $\pm 30$). This happens since it is expected that Object A will win against Object B. If for some reason, Object B wins against Object A, the increase in rating will be significant to help "course-correct".


<a name="section4"/>

## 4. APIs developed

### 4.1. GET `/GetPlayers`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/getPlayerDiagram.png" alt="GetPlayers Diagram"/>
</p>

This API is used to get the list of `Object of Comparison`, as well as the number of votes the room has got so far (`votesCount`). We have 2 parameters, `roomId` being the compulsory parameter and `sorted` is optional. This API will trigger a Lambda function that will get the room details from MongoDB table. If the user sets `sorted` to be true (mainly used in the game scenario and leaderboard), the response will be sorted according to the `playerRating` value.

```vbnet
REQUEST:
GET /getPlayers?<Parameter>

Parameter:
- roomId: <string>
- sorted: <boolean> [Optional]
```

```vbnet
RESPONSE:
{
  roomId:  "<roomId>",
  votesCount: 35,
  players: [
    {"playerId":"<uuid>", "playerRating":985, "name":"Boeing 747", "image":"<random image link>", "status":"ACTIVE"},  
    {"playerId":"<uuid>", "playerRating":1105, "name":"Airbus A380", "image":"<random image link>", "status":"ACTIVE"}, 
    {"playerId":"<uuid>", "playerRating":1001, "name":"Airbus A350", "image":"<random image link>", "status":"ACTIVE"},
    {"playerId":"<uuid>", "playerRating":909, "name":"Boeing 737", "image":"<random image link>","status":"ACTIVE"},
  ]
}
```

---

### 4.2. POST `/CreateNewRoom`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/createNewRoom.png" alt="GetPlayers Diagram"/>
</p>

This API creates a room with a specified `roomId` and a list of players. When a new room is created, every player starts with a score of 1000, which is managed by the front end. The front end also validates whether a room with the given `roomId` already exists. When the user enters a `roomId`, a request is sent to the /getPlayers API to retrieve the current list of `Object of Comparison`. If no such room exists, the API is then called.

```vbnet
REQUEST:
POST /CreateNewRoom

Request Body:
{
  roomId: "12349"
  players:
    [
      {"playerId":"<uuid>", "playerRating":1000, "name":"Boeing 747", "image":"<random image link>", "status":"ACTIVE"},  
      {"playerId":"<uuid>", "playerRating":1000, "name":"Airbus A380", "image":"<random image link>", "status":"ACTIVE"}, 
      {"playerId":"<uuid>", "playerRating":1000, "name":"Airbus A350", "image":"<random image link>", "status":"ACTIVE"},
    ]
}
```

```vbnet
RESPONSE:
Status Code: 200
{"acknowledged":true,"insertedId":"<random-id>"}
```

---

### 4.3. POST `/EditRoom`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/editRoom.png" alt="GetPlayers Diagram"/>
</p>

This API sets the content of the `roomId` to be `players`. Essentially, all the logic of setting `DELETED` status is handled in the front-end. The back-end will loop through the changes, and make the change according to the `changes` from the request.

```vbnet
REQUEST:
POST /EditRoom

Request Body:
{
  roomId: "12349"
  changes:
    [
      {
          "playerId": "<uuid>",
          "change": "DELETE"
      },
      {
          "playerId": "<uuid>",
          "change": "EDIT",
          "values": {
              "name": "Porsche Macan"
          }
      },
      {
          "playerId": "<uuid>",
          "change": "CREATE",
          "values": {
              "playerId": "<uuid>",
              "name": "BMW iX",
              "image": "https://hips.hearstapps.com/hmg-prod/images/2023-bmw-ix-m60-108-1653422436.jpg?crop=0.728xw:0.820xh;0.160xw,0.0264xh&resize=768:*",
              "playerRating": 1000,
              "status": "ACTIVE"
          }
      }
    ]
}
```

```vbnet
RESPONSE:
Status Code: 200
"Success"
```

#### 4.3.1 Why do I decide to mark the deleted `Object of Comparison` (soft delete) rather than actually deleting the `Object of Comparison` (hard delete)?
Consider a scenario with two users, User A and User B. User A caches data for one round, which can cause problems if User B deletes an `Object of Comparison` in the room that User A is in. If this deletion occurs, User A's message request in the SQS queue could fail because the Lambda function will try to access the now-deleted `Object of Comparison` from MongoDB.

To avoid this issue, we can mark the `Object of Comparison` as deleted instead of immediately removing it. This way, the `UpdateRating` Lambda function can still update User A’s SQS message to update the score, even though User B has deleted the `Object of Comparison`. In the following round, User A will receive the most recent data and will no longer see the deleted `Object of Comparison`.

#### 4.3.2 Why am I just keeping the changes log and not overwriting everything?

Let's consider two users, User A and User B, who attempt to access the edit room functionality simultaneously. If the system simply overwrites values, the final value retained in the system will be from the user who submits their editRoom request last, resulting in the earlier submission being overwritten.

If we implement a change log system, we can handle these requests in parallel without losing any information. Each request can be recorded as a distinct entry in the change log, capturing all changes made by both users. This way, the system does not need to wait for one request to complete before processing the next one, thereby potentially saving time and improving efficiency. Additionally, using a change log allows for better tracking of changes and conflict resolution, ensuring that all user edits are preserved and can be merged or reviewed as needed.

#### 4.3.3. Lambda functions to edit the data to MongoDB

```js
async function deletePlayer(db, roomId, playerId) {
    /*
      Executed when we need to delete a particular player with ID === playerId. We are simply setting
      the "status" field to be "DELETED" to mark the item as deleted. Since we need to make
      alteration inside an array (which is a member attribute of the "Room" collection),
      we need to use arrayFilter to make adjustment to edit specific member of the array.
    */

    const result = await db.collection("Room");
        .updateOne(
            {"roomId" :  roomId},
            {$set: {
                'players.$[x].status': "DELETED",
            }},
            {arrayFilters: [
                {"x.playerId": playerId},
            ]}
        );
    return result;
}

async function editPlayer(db, roomId, playerId, values) {
    /*
      Executed when we need to edit a particular player with ID === playerId.
      Each changed field is contained in the values as key-value pairing. We
      first need to extract the key-value pairing, and adjust it to mimic
      our MongoDB query.
    */

    // adjusting the key-value pairing of values
    let editQuery = {};
    Object.keys(values).forEach(function(key) {
        editQuery[`players.$[x].${key}`] = values[key];
    })

    // executing MongoDB function using the adjusted values
    const result = await db.collection("Room")
    .updateOne(
        {"roomId" :  roomId},
        {$set: editQuery},
        {arrayFilters: [
            {"x.playerId": playerId},
        ]}
    );
    return result;
}

async function createPlayer(db, roomId, values) {
    /*
      Executed when we need to create a particular player. We are
      simply pushing the value to our players attribute, which
      happens to be an array.
    */

    const result = await db.collection("Room")
        .updateOne(
            {"roomId" :  roomId},
            {$push: {players: values}}
        );
}
```

---

### 4.4. POST `/UpdateRating`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/updateRating.png" alt="GetPlayers Diagram"/>
</p>

When the user submits the `/updateRanking` POST API call, API Gateway then triggers the `SendToSQSQueueFunction` Lambda function upon receiving the POST request. This Lambda function processes the initial request and sends a message to the SQS queue. The SQS queue acts as a buffer, decoupling the request processing from the response. Subsequently, Worker `UpdateRatingFunction` Lambda functions poll the SQS queue, process the messages, update the ratings on MongoDB, as well as incrementing the `votingCount` of the room by 1.

I opted to cache the `playerRating` for all `Object of Comparison` on the front end. This means the Worker `UpdateRatingFunction` Lambda function won't need to constantly query MongoDB for ratings, reducing the number of database reads. However, this also means that until the round ends, comparison objects will retain their ratings as they were when the user first loaded the game on the front end. Once all pairs have been compared, the front end will invoke the `/getPlayers` API, resetting the cache and providing the user with accurate information at the time of the API request submission.

#### 4.4.1. `UpdateRating` Lambda Function (Worker Function) Logic

I've chosen to calculate the difference in `Object of Comparison`'s ratings and delegate the task of incrementing the MongoDB value to the SQS Worker `UpdateRatingFunction` Lambda. With users having a cache of player ratings, we can utilize it to compute the difference between old and new ratings for both players. This difference is then incremented to MongoDB by the SQS Worker `UpdateRatingFunction` Lambda.

Why opt for incrementing/decrementing the difference instead of directly updating to the new rating in the Worker `UpdateRatingFunction` Lambda? Consider this scenario: two identical SQS messages arrive simultaneously from different users playing at the same time.

```json
{
  "roomId": "12345",
  "losingPlayerId": "1",
  "losingPlayerRating": 1500,
  "winningPlayerId": "2",
  "winningPlayerRating": 1000
},
{
  "roomId": "12345",
  "losingPlayerId": "1",
  "losingPlayerRating": 1500,
  "winningPlayerId": "2",
  "winningPlayerRating": 1000
}
```

According to the earlier Elo-Score calculation, `Object of Comparison` with ID `2` should gain an additional `30 + 30 = 60` points, resulting in a rating of `1060`. However, if we directly set the value (instead of using increment) from the Worker Lambda, we'll only obtain `1030`, which was the last rating value written by the last message from the queue (as values get overwritten). Since the calculation is independent for each message using the information provided from the JSON, using an increment method ensures that the score can be updated to reflect all votes. This approach enables us to still obtain a close-to-accurate rating from the cached values.

#### 4.4.2. Lambda function to increment the values of the 2 players
```js
async function updateRankingFunction(db, winningPlayer, losingPlayer, roomId) {
    // get the difference in rating using the function getNewRatingAddition
    const differenceInRating = getNewRatingAddition(winningPlayer.rating, losingPlayer.rating);

    // update to MongoDB, using inc, and setting the increment of the 2 values directly
    // to maintain atomicity
    const result = await db.collection("Room").updateOne(
        {"roomId" :  roomId},
        {$inc: {
            'players.$[x].playerRating': differenceInRating[0],
            'players.$[y].playerRating': differenceInRating[1],
            'votesCount': 1
        }},
        {arrayFilters: [
            {"x.playerId": winningPlayer.id},
            {"y.playerId": losingPlayer.id}
        ]}
    );

    return result;
}

```


```vbnet
REQUEST:
POST /UpdateRanking

Request Body:
{
  roomId: "12345",
  losingPlayerId: "3",
  losingPlayerRating: 909,
  winningPlayerId: "1"
  winningPlayerRating: 1105
}
```

```vbnet
RESPONSE:
Status Code: 200
{"message":"Message sent to SQS","messageId":"<randomId>"}
```

#### 4.4.3. Architectural Alternative 1: Direct Handling (Not Recommended)

The other approach is to handle the update rating on the lambda which is directly invoked by the `/updateRating` API.

The client sends a POST request to the `/updateRating` API. The API Gateway receives this request and directly triggers a Lambda function. This Lambda function immediately handles the request, performs the necessary processing to update the rating, and returns a response to the client.

**Pros**:
  - Simpler architecture with fewer components.
  - Lower latency with immediate processing.
  - Potential cost savings from fewer services being used.

**Cons**:
  - Scalability issues and potential bottlenecks under high load.
  - Less fault tolerance and resilience without SQS.
  - More complex load management within a single Lambda function.


#### 4.4.4. Chosen Alternative 2: Using SQS Approach
**Pros**:
  - Better scalability and load handling due to decoupling.
  - Improved fault tolerance with message retry mechanisms.
  - Resilience to traffic spikes, as SQS can buffer requests.

**Cons**:
  - Added complexity with more components to manage.
  - Potentially higher latency due to asynchronous processing.
  - Additional costs associated with SQS and multiple Lambda invocations.

#### 4.4.5. Verdict
Since I want to make the system scalable, I decided to use the SQS approach.

---

### 4.5. GET `/GetStatistics`

<p align="center">
  <img src="https://github.com/mattboentoro/ThisOrThatDocumentation/blob/main/pictures/getStatistics.png" alt="GetPlayers Diagram"/>
</p>

This API gets the room with the most number of `votesCount`, as well as accumulating the number of votes across all the rooms, and returning it under `totalVotesCount`. This leverages MongoDB functionality that it allows me to perform `aggregate` and got the sum of the attribute member `votesCount`.

```js
// MongoDB snippet from Lambda function

// Aggregating the sum from $votesCount

const pipeline = [
    { $group: { _id: null, total: { $sum: "$votesCount" } } }
];
const aggCursor = db.collection("Room").aggregate(pipeline);

// Getting the room with the greatest number of votesCount

const aggCursor = db.collection("Room")
                      .find()
                      .sort({'votesCount': -1})
                      .project({ roomId: 1, votesCount: 1 })
                      .limit(1);
```

```vbnet
REQUEST:
POST /GetStatistics
```

```vbnet
RESPONSE:
Status Code: 200
{
  "totalVotesCount":50,
  "mostPopularRoom":
      {
        "_id":"66569ff357a1050cb28c9709",
        "roomId":"123456",
        "votesCount":26
      }
}
```

---

<a name="section5"/>

## 5. Database Schema
```json
{
  "roomId": "<string>",
  "players": [
    {
      "playerId": "<string> [uuid]",
      "image": "<string>",
      "name": "<string>",
      "playerRating": "<int>",
      "status": "<string> [ACTIVE | DELETED]",
    }
  ],
  "votesCount": "<int>"
}
```

<a name="section6"/>

## 6. How do we get all players combinations to be paired?
I used slightly tweaked (Round Robin)[https://github.com/tournament-js/roundrobin?tab=readme-ov-file ] algorithm to get a pairing list of two `Object of Comparison`, such that all `Object of Comparison` will get to meet one another on one round. This is done on the front end, after the user gets the room information from `/getRooms` API.

<a name="section7"/>

## 7. Why do I use MongoDB compared to other database option?
- Seamless integration with Node.js-backed Lambda.
- Supports data aggregation that I needed for `/getStatistics`.
- Supports `$inc` increment function that is atomic to update various attribute members.
- Better table scan performance.


<a name="section8"/>

## 8. Future works

- [x] <s>Use React-Router to allow direct access to all pages from the URL (like `<URL>/play/<roomId>`).</s>
- [ ] Support room deletion.
- [x] <s>Support edit individual `Object of Comparison` after added.</s>
- [x] <s>Validate the API calls on the front-end, and return error message accordingly.</s>
- [x] <s>Support statistics (vote count), and show the number of votes on the home screen.</s>
- [ ] Support versioning for `updateRoom`. Essentially, adding a field on the table namely `version`, which gets incremented every time `updateRoom` is called successfully. If the room is in version 6, and the request comes in based on version < 6, we can reject the request.
- [ ] Make an explore page for `roomId`.


<a name="section9"/>

## 9. Helpful links that helped me during the project:

- https://github.com/tournament-js/roundrobin?tab=readme-ov-file 
- https://www.mongodb.com/docs/manual/reference/operator/update/inc/
- https://www.mongodb.com/community/forums/t/mongodb-update-a-value-in-array-of-object-of-array/208302
- https://www.mongodb.com/developer/products/atlas/serverless-development-lambda-atlas/
- https://sadam-bapunawar.medium.com/add-and-remove-form-fields-dynamically-using-react-and-react-hooks-3b033c3c0bf5
- https://stackoverflow.com/questions/36734201/how-to-convert-numbers-to-million-in-javascript
- https://stackoverflow.com/questions/17044587/how-to-aggregate-sum-in-mongodb-to-get-a-total-count
