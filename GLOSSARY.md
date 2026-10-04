# Glossary

Domain terms for Minilagr. Use these words in code, issues and docs. The Norwegian term is what users see in the UI.

| Term | Norwegian (UI) | Code | Meaning |
| --- | --- | --- | --- |
| Platform owner | Plattformeier | — | Us (Minilagr). Creates and bills operators and sees operations and errors across all of them. |
| Operator | Operatør | `operators` | A business that runs one or more self-storage facilities on Minilagr. The tenant: all data is scoped by `operator_id`. |
| Operator owner | Operatøreier | — | The person who owns an operator. Sets up locations, units and prices, and sees finances and reports. |
| Operator staff | Operatøransatt | — | Caretaker or customer service at an operator. Handles customers, grants access, logs damage and maintenance. |
| Location | Lokasjon | `locations` | One physical facility with an address, opening hours and access rules. |
| Unit | Bod | `units` | One rentable storage unit at a location. Has a number, size (m² and m³), floor, type (indoor, outdoor, climate-controlled) and status. |
| Customer | Sluttkunde | `customers` | A private person or a business (with an organisation number) that rents a unit. |
| Lease | Leieavtale | `leases` | The signed rental agreement between a customer and an operator for a unit. Signed with BankID. |
| Payment | Betaling | `payments` | A charge to the customer: the first payment at booking, then monthly charges by card or Vipps. |
| Access code | Adgangskode | `access_codes` | A personal code that opens the gate and the customer's own unit, and nothing else. |
| Lock adapter | Låsadapter | — | The single interface every lock vendor implements: create code, change code, block code. |
| Dunning | Purring | — | Reminders with deadlines after a failed payment, ending in blocked access. |
| Waitlist | Venteliste | — | Customers waiting for a unit size that is sold out. They get an offer automatically when one frees up. |
| Customer page | Min side | — | The customer's self-service page: units, code, payments, receipts, card update, cancellation. |
| Operator panel | Operatørpanel | — | Where operators run their facilities: setup, prices, daily operations, finances. |
| Platform admin | Plattformadmin | — | Our cross-operator view: operator list, health checks, log in as an operator for support. |
