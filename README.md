# ParkEasy — Parking Management System (WBD I Midterm)

A browser-based parking management website backed by a REST API built with **plain Node.js**. It intentionally does not use Express, MongoDB, Prisma, or a frontend framework so it remains aligned with the WBD I midterm requirements. Data is stored in JSON files in `data/` and activity events are recorded in `activity.log` using Node's EventEmitter.

## Start the project

```bash
npm install
npm start
```

Open the website at **http://localhost:3000/app**. The API root at **http://localhost:3000/** returns JSON as required by the assignment.

For development with automatic restart:

```bash
npm run dev
```

Run the assignment self-check:

```bash
npm run check
```

The self-check runs on port `3999` with a temporary data directory and currently contains 62 checks.

## Website modules

- **Dashboard:** live capacity, available/occupied counts, utilization, seven-day activity, recent sessions, and facility summary.
- **Live parking lot:** visual parking bays, available/occupied filters, vehicle assignment, and check-out.
- **Vehicles:** register, search, edit, and delete vehicles; prevents deleting a vehicle that is currently parked.
- **Parking slots:** add, search, edit, and delete slots; prevents deleting an occupied slot.
- **Zones:** create and manage named parking areas.
- **Parking history:** view active/completed sessions, check vehicles out, search records, and export CSV.
- **Reports:** session status, average completed stay, and check-in time distribution; print-friendly layout.
- **Demo data:** from an empty dashboard, use **Load demo data** to create sample zones, slots, vehicles, and active sessions for a classroom demonstration.

The interface is responsive for desktop and mobile. All displayed operational data is loaded from the Node.js API; it is not a static mockup. Reports are calculated from saved parking sessions. This assignment version does not implement user authentication, payment processing, or actual revenue collection.

## API routes

| Resource | Routes | Fields |
|---|---|---|
| Root | `GET /` | API status and resource paths |
| Slots (ITEM) | `GET/POST /slots`, `GET/PUT/DELETE /slots/:id` | `{id, name, note}` |
| Vehicles (PERSON) | `GET/POST /vehicles`, `GET/PUT/DELETE /vehicles/:id` | `{id, name, phone}` |
| Zones (EXTRA) | `GET/POST /zones`, `GET/PUT/DELETE /zones/:id` | `{id, name, note}` |
| Parkings (LINK) | `GET/POST /parkings`, `GET /parkings/:id`, `PUT /parkings/:id/end`, `DELETE /parkings/:id` | `{id, itemId, personId, startedAt, endedAt}` |

For a parking record, `itemId` is the slot ID and `personId` is the vehicle ID. A slot cannot be occupied by two active sessions at once. `endedAt` is `null` until check-out.

## Assignment implementation

- Modules use `require()` and `module.exports`.
- The server uses `http.createServer()`; no Express.
- JSON persistence uses `fs.promises` and `async/await`.
- The API returns JSON and appropriate status codes (`200`, `201`, `400`, `404`, `405`, `409`, `500`).
- Missing data files are treated as empty lists on first run.
- Create/update/delete operations emit EventEmitter events; `activity.js` appends them to `activity.log`.
- JSON files are written through a temporary file followed by rename to reduce the chance of partial writes.

## Example API calls

```bash
curl http://localhost:3000/
curl http://localhost:3000/slots
curl -X POST http://localhost:3000/slots -H "Content-Type: application/json" -d "{\"name\":\"A1\"}"
curl -X POST http://localhost:3000/vehicles -H "Content-Type: application/json" -d "{\"name\":\"Toyota 2A-1234\",\"phone\":\"012345678\"}"
curl -X POST http://localhost:3000/parkings -H "Content-Type: application/json" -d "{\"itemId\":1,\"personId\":1}"
curl -X PUT http://localhost:3000/parkings/1/end
```

## Project structure

```text
parking-api/
├── public/
│   └── index.html       # ParkEasy website (HTML, CSS, JavaScript)
├── resources/
│   ├── slots.js         # ITEM resource
│   ├── vehicles.js     # PERSON resource
│   ├── parkings.js     # LINK resource
│   └── zones.js        # EXTRA resource
├── lib/
│   ├── events.js
│   ├── respond.js
│   └── static.js
├── data/                # JSON data files created at runtime
├── activity.js
├── router.js
├── server.js
├── check.js
└── package.json
```
