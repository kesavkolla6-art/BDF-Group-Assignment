# BookNook Cafe: Board Games API

**Section:** GAI2A · **LG 7** (Group 7 of 15) · **Card:** 322 of 375 · **Theme:** 22 of 30

| S. No. | Student Name | Roll Number | Individual Contribution |
| ------ | ------------ | ----------- | ----------------------- |
| 1 | G.S.Rithwik | 9036 | Server setup and routes |
| 2 | C.H Venu | 9023 | Database creation and queries |
| 3 | K.Kesav | 9068 | REST API endpoints |
| 4 | Nikitha | 9035 | Validation and search |
| 5 | Dharani | 9043 | Twist rule: limit of 3 active bookings and the return flow |
| 6 | Monica Asmitha | 9061 | Postman testing and README documentation |

**Live link:** https://bdf-group-assignment-2.onrender.com/

## 1. Project Overview

BookNook Cafe: Board Games API is a backend application developed using Node.js, Express.js and SQLite.

At the cafe, guests pick board games from a shelf. Pieces go missing and the cafe cannot see which games are in use.

This project provides REST APIs to manage guests, games and bookings. Every time a guest takes a game it is recorded as a booking, and the booking stays active until the game is returned. The cafe can always see which games are out and who has them.

## 2. Technologies Used

- Node.js
- Express.js
- SQLite 
- JavaScript
- Postman

## 3. Team Members and Contributions

| S. No. | Student Name | Roll Number | Individual Contribution |
| ------ | ------------ | ----------- | ----------------------- |
| 1 | G.S.Rithwik | 9036 | Express setup, routes, status codes, error handling. |
| 2 | C.H Venu | 9023 | The 3 tables, foreign keys, seed data, the join and in_use queries. |
| 3 | K.Kesav | 9068 | CRUD for games and guests, the join endpoint for bookings. |
| 4 | Nikitha | 9035 | Input checks with clear messages, search by players, name and in_use. |
| 5 | Dharani | 9043 | Limit of 3 active bookings per guest, the return flow, and verified Twist Tests 1 to 3. |
| 6 | Monica Asmitha | 9061 | Postman collection (38 requests, 42 checks) and this README. |

## 4. How to Run the Project

**Step 1: Clone the repository**

```
git clone https://github.com/kesavkolla6-art/BDF-Group-Assignment.git
```

**Step 2: Open the project folder**

```
cd BDF-Group-Assignment
```

**Step 3: Install dependencies**

```
npm install
```

**Step 4: Start the server**

```
npm start
```

For development with automatic restart use `npm run dev`.

The server will run at `http://localhost:3000`.

The SQLite database (`booknook.db`) is included with sample data (5 guests, 8 games, 3 bookings). To rebuild it from scratch run `npm run seed`. The tables are also created automatically when the server starts. `PORT` is optional (see `.env.example`).

## 5. Database Design

The project uses three tables.

### Guests Table

| Column | Description |
| ------ | ----------- |
| id | Guest ID |
| name | Guest name |
| phone | Guest phone (optional) |

### Games Table

| Column | Description |
| ------ | ----------- |
| id | Game ID |
| name | Game name (unique) |
| players | Number of players |

### Bookings Table

| Column | Description |
| ------ | ----------- |
| id | Booking ID |
| guest_id | Guest who took the game (foreign key) |
| game_id | Game that was taken (foreign key) |
| booked_at | When the game was taken |
| returned_at | When it was returned (empty while the booking is active) |

## 6. API Endpoints

### Games

| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| GET | /games | Get all games (with `in_use`). Search: `?players=4`, `?search_q=ch`, `?in_use=true` |
| GET | /games/:id | Get a game by ID |
| POST | /games | Add a game |
| PUT | /games/:id | Update a game |
| DELETE | /games/:id | Delete a game (409 if it has bookings) |

### Guests

| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| GET | /guests | Get all guests. Search: `?search_q=asha` |
| GET | /guests/:id | Get a guest by ID |
| POST | /guests | Add a guest |
| PUT | /guests/:id | Update a guest |
| DELETE | /guests/:id | Delete a guest (409 if they have bookings) |

### Bookings

| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| GET | /bookings | Get bookings with guest name and game name (JOIN). Filters: `?status=active\|returned`, `?guest_id=` |
| GET | /bookings/:id | Get one booking with guest and game names |
| POST | /bookings | Create a booking (409 "Limit of 3 reached" on the 4th active one) |
| POST | /bookings/:id/return | Return a game (409 "Already returned" if done twice) |
| DELETE | /bookings/:id | Delete a booking |

Every error response has the same shape: `{ "message": "..." }`.

## 7. Validation and Search

The application includes input validation to ensure that the API receives valid data.

- Game name is required and must be text with a maximum length of 50 characters.
- Game players is required and must be a whole number from 1 to 20.
- Guest name is required. Guest phone is optional text.
- Booking `guest_id` and `game_id` are required positive whole numbers, and the guest and game must exist (404 otherwise).
- ID in the path must be a positive whole number.
- A game name that already exists returns 409.
- Search: games by players, by part of the name and by whether they are in use; guests by name; bookings by status and guest.
- All values go into SQL through `?` placeholders.

## 8. Special Feature: Three at a Time

A guest can hold at most 3 active bookings. The 4th attempt gets **409** with the message **"Limit of 3 reached"**. Returning a game frees a place.

| Test | Action | Expected |
| ---- | ------ | -------- |
| 1 | Create 3 bookings for Asha | 201, 201, 201 |
| 2 | Create a 4th booking for Asha | 409, Limit of 3 reached |
| 3 | Return one, then create a new booking | 201 |

- `POST /bookings/:id/return` sets `returned_at` and returns 200.
- Returning the same booking again returns 409 "Already returned". An unknown booking returns 404.
- The limit is enforced in one SQL statement (insert only if the guest has fewer than 3 active bookings), so two requests arriving together cannot both pass. A test with 8 simultaneous requests gave exactly the free places and 409 for the rest.
- Each guest is counted separately.

## 9. Postman Testing Screenshots

The APIs were tested using Postman. The following screenshots demonstrate the twist (limit of 3 and the return flow) and the overall test results.

### Screenshot 1: Create Booking 1 for Asha

`POST /bookings` with `{"guest_id": 1, "game_id": 1}`

Expected result: 201 Created.

![Create booking 1](screenshots/01_Test1_Booking_1.png)

### Screenshot 2: Create Booking 2 for Asha

Expected result: 201 Created.

![Create booking 2](screenshots/02_Test1_Booking_2.png)

### Screenshot 3: Create Booking 3 for Asha

Expected result: 201 Created.

![Create booking 3](screenshots/03_Test1_Booking_3.png)

### Screenshot 4: Fourth Booking for Asha

Expected result: 409 Conflict with the message "Limit of 3 reached".

![Fourth booking refused](screenshots/04_Test2_Limit_Reached.png)

### Screenshot 5: Return a Booking

`POST /bookings/:id/return`

Expected result: 200 OK. `returned_at` is set.

![Return booking](screenshots/05_Return_Booking.png)

### Screenshot 6: Return the Same Booking Again

Expected result: 409 Conflict with the message "Already returned".

![Return again](screenshots/06_Return_Again.png)

### Screenshot 7: New Booking After a Return

Expected result: 201 Created.

![New booking after return](screenshots/07_Test3_New_Booking.png)

### Screenshot 8: Postman Collection Runner Results

The complete Postman collection was executed using the Collection Runner.

Expected result: 42 checks passed and 0 failed.

![Postman test results](screenshots/08_Postman_Test_Results.png)

## 10. Postman Collection

The Postman collection contains 38 requests covering different API functionalities.

The requests are organized into the following categories:

- Guests
- Games (CRUD, validation and search)
- Bookings, JOIN and the limit of 3 (Tests 1 to 3, return flow)

The collection file is available at:

`postman/BookNook.postman_collection.json`

The collection uses the following variable:

| Variable | Value |
| -------- | ----- |
| baseUrl | http://localhost:3000 |

To run it: import the file in Postman, start the server, open the collection and click **Run**. A setup request adds a fresh guest, so it can be run again and again.

## 11. Project Structure

```
BDF-Group-Assignment/
│
├── routes/
│   ├── guests.js
│   ├── games.js
│   └── bookings.js
├── app.js
├── database.js
├── validators.js
├── utils.js
├── schema.sql
├── seed.sql
├── seed.js
├── booknook.db
├── package.json
│
├── postman/
│   └── BookNook.postman_collection.json
│
├── screenshots/
│   ├── 01_Test1_Booking_1.png
│   ├── 02_Test1_Booking_2.png
│   ├── 03_Test1_Booking_3.png
│   ├── 04_Test2_Limit_Reached.png
│   ├── 05_Return_Booking.png
│   ├── 06_Return_Again.png
│   ├── 07_Test3_New_Booking.png
│   └── 08_Postman_Test_Results.png
│
├── documents/   (individual Word document of each member)
│
└── README.md
```

## 12. Conclusion

The BookNook Cafe: Board Games API provides a simple way to manage guests, games and bookings. It supports CRUD operations, validation, search and JOIN queries. The limit of 3 active bookings per guest, with a return flow that frees a place, lets the cafe see which games are in use and who has them. All API functionalities were tested using Postman.
