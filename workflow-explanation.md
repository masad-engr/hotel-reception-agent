# Workflow explanation

## Entry point and agent

`When chat message received` accepts a chat message and passes it to `Hotel Concierge Agent`. The agent receives the chat input and is configured with an OpenAI Chat Model (`gpt-5-mini` in the supplied export) and Postgres Chat Memory.

## Tools connected to the agent

| Node | Intended responsibility |
|---|---|
| `Hotel Rooms List` | Read the room inventory from Google Sheets. The prompt identifies room ID, room type, capacity, nightly price, and description as relevant fields. |
| `List Of Reservation` | Read reservation records to verify availability and match reservation ID with guest email. |
| `Create New Reservations` | Append a confirmed reservation to the reservation sheet after the required checks and guest confirmation. |
| `modify_reservation` | Update a reservation record matched by reservation ID. |
| `Cancel Reservations` | Set a reservation status to `cancelled`. |
| `Hotel FAQs` | Retrieve hotel FAQ content from Google Sheets. |
| `Hotel info` | Retrieve general hotel information from Google Sheets. |
| `Gmail Confirmation sender` | Send a reservation email after a create, modify, or cancel action succeeds, according to the agent instructions. |

## Reservation rules encoded in the prompt

- A confirmed reservation blocks the same room when the existing stay overlaps the requested stay: existing check-in is before requested check-out, and existing check-out is after requested check-in.
- Cancelled reservations do not block a room.
- Requested guest count must not exceed room capacity.
- Check-in must be before check-out; dates must be validated rather than silently changed.
- Total price is calculated from the selected room's authoritative nightly rate multiplied by the number of nights.
- The guest must explicitly confirm the complete booking summary before a create, modification, or cancellation action.
- Reservation details require both the reservation ID and matching guest email.
- Requests that cannot be verified, and complaints/refunds/payment/safety issues, should be handed to hotel staff.

## Configuration that remains

This public export intentionally does not contain credential bindings or private spreadsheet references. After import, configure your own Google Sheets document and tabs, OpenAI credentials, Gmail credentials, and Postgres Chat Memory connection.

The export describes intended behavior, not proof of successful execution. Test each path with dummy records and inspect execution logs before relying on it.
