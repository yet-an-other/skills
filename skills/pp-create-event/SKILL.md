---
name: pp-create-event
description: Use this skill if user wants to create an event(s) in google calendar out of provided file(s) or text description. Use it then user asks "add an event", "create event", "добавь в календарь", "создай в календаре"
---

## Preparation
- Extract all information from the provided files
- Use subagents to parse pdf, docx, OCR images or any other extracting text activity in order to save context
- If there is a similar event at the same time, ask user if this is a duplication, and do not add event if so
- If the PDF contains multiple events (flights/legs/sessions), make sure to capture all of them.
- Define if some of the provided files (or a part of) is worth to store in the event as an attachment. It could be a QR codes on ticket, map, scheme or some other meaningful information that could not be converted to a text or stored in an event note for some reason.
- If some information is not available directly from data provided, but may be useful, try to find it in internet. E. g. departure and arrival terminals can be found using flight number.

## Constructing an event
- Use Google API to create a calendar event
- When creating Google Calendar events via shell/terminal, always use execute_code with real newline characters in Python strings, combined with shlex.quote(). Never pass \n inside double-quoted bash strings via terminal(), because bash does not interpret \n as newline in double quotes — it passes literal backslash-n to the API, resulting in garbled event descriptions.

## Formatting
- Prefix event summaries with an emoji (airplane emoji ✈️ for flights, train for train, ticket for theater/kino, fork and knife emoji for restaurants and so on).
- Use green color for transportation events
- Use multiline notes to provide an event info, separating useful information
- Do NOT put links in the text on the same file that in attachment
- Do not look for previous events of similar type as an example, use examples below instead 

### Examples 

#### Flight event
```
Flight OS 368 (Austrian Airlines)
CDG, Terminal 1 -> VIE, Terminal 3
Class: Economy (T)
Seat: 10C
Baggage: 1PC 1x10 Kg
Departure: 20:15
Arrival: 22:10
Duration: 1h 55m
PNR: ONSQLU
Ticket: Xxxx
```

## Attachments
- If there is any attachment worth to store (and only if it is worth to store), use "CalendarAttachments" folder on user's google drive to upload and store attachments.
- Use date and time of the event starting from a year as a prefix to an attachment's filename for sorting.
- Add a link to an attachment in the event as part of the event data in a google calendar format.
