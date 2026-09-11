# RoomReserve Critique

Fill in each section. One section per milestone. Keep it short and specific. Point at
files and methods, not adjectives.

---

## Milestone 1: The design as it is

Describe the system as the code actually builds it.

**Data model.**
A booking is 5 pieces of information: 'room, date, start, end, user'. Requests contain strings, and start/end times are converted to long minutes. Storage splits a booking across slotsByRoomDate (room/date → list of long[] intervals) and bookerBySlot (room/date/start/end → user). There inn't a Booking class.

**Operations.** What can a caller do, and what goes in and out?
- book a room: booking information -> string confirming action
- cancel a booking: booking information -> string confirming action
- reschedule booking: old booking & new time -> string confirming action
- list bookings for a data: roomID & date -> string with a list of bookings

**Structure.** What classes exist, what does each own, and who holds a reference to whom?
- BookingPolicy: Rules (ex. buisness hours, no overlap, etc.)
- InMemoryStore: Maintains list of room/dates, lotsByRoomDate
- RequestHandler: Add bookings, cancel bookings, etc. holds references to InMemoryStore
- ReservationApp: Demo entry point, holds reference to RequestHandler

**The no-double-booking invariant.**

BookingPolicy.validate() checks end-after-start, business hours, maximum duration, and overlaps with existing bookings, but nothing calls it. There is another check in addSlot, but that one just checks if its exact, not if it overlaps. 

Finally, there are checks in RequestHandler for addBooking  which loop through the bookings for the appropraite room/date and check if there is any overlap. Importantly, rescheduleBooking() does not check overlaps. Business hours and maximum duration are also unenforced in both creation and rescheduling.

**Reschedule**
ReservationApp.main() calls RequestHandler.rescheduleBooking(). The handler converts the times to longs, checks that the new end follows the new start, and retrieves the original user through store.bookerFor(). Then it calls removeSlot() and addSlot(). Moving a booking to 09:30–10:30 when another booking occupies 09:00–10:00 therefore creates an overlap.

---

## Milestone 2: Two design problems

### Problem 1

**The problem.** 
Representational Gap: There isn't a concrete datastructure for a room bookings which would  lead to future complications. For example, the data, start, and end times should be in an explicit datastructure that track time. The RequestHandler just takes strings and doesn't check if the given strings are consistent and verified.

**Where in the code.** 
RequestHandler: Createbooking, CancelBooking, etc.

**What it makes expensive.**
How does the code know that the date written like: January 1st, 2001 is the same as 1/01/2001. There isn't any normalization/verification for the dates/times which would lead to the code breaking for giving incorrect results. 

### Problem 2

**The problem.** 
Misplaced responsibility: RequestHandler checks booking rules, ex: no overlap rather than using BookingPolicy to handle all the rules.

**Where in the code.**
RequestHandler: createBooking()
BookingPolicy: rescheduleBooking()

**What it makes expensive.**
If a rule changes, (ex: bookings can be consectutive a.k.a one booking can end at the same time another starts) then you would have to change the rules everywhere which could leads to errors instead of just need to change it in BookingPolicy and ensure the change is applied consistently. 

---

## Milestone 3: Two alternative decompositions

### Alternative A

**The decomposition.** What are the pieces, what does each own, and where do the rules
live?

The first alternative is to just store the bookings as a flat list of some specified datastructure that represents the bookings. The all the rules would live in BookingPolicy, and whenever a new booking was added it check through all the bookings to make sure there wasn't any overlap, the fell in business hours, etc. 

InMemoryStore, RequestHandler, and ReservationApp would keep their parts.

**One tradeoff.** If there are a lot of bookings (ex. over 10k) it becomes slow and inefficient to check if a booking follows the rules.

### Alternative B

**The decomposition.**

The second alternative is to have two nested hasmaps:
- hashmap by date -> hashmap by rooms
- hashmap by rooms -> hashmap by date
Both of these hashmaps point to the same list of bookings this way it is easier to sort bookings by date or room number. The all the rules would live in BookingPolicy.
InMemoryStore, RequestHandler, and ReservationApp would keep their parts.

**One tradeoff.**There would be extra complications by having 3 copies of the same data, like having lots of overhead especially if list of bookings grows to a large number.

### Preference

I would pick Alternative B because it would allow for more effective checking and you would be able to more easier sort/search through the data.
