# Week 2 Self-Check + Checkpoints - Ratu Faiha Salsabilla (24/532756/PA/22533)

## Checkpoint 1
A screen-based address I might initially think of is 'GET /booking-page', because the user needs a page to view and make bookings. However, this should be split into actual resources such as '/courts' and '/bookings', since courts and bookings are persistent resources that can be identified and changed independently.

## Checkpoint 2
For my badminton booking system, a booking can change from 'pending' to 'confirmed'. Following rule 5, I would represent this as 'POST /bookings/{bookingId}/confirmation'. The verb version: 'POST /bookings/{bookingId}/confirm' is worse because it uses an action as the URI instead of representing the state transition as a noun-based sub-resource.

## Checkpoint 3 
For 'GET /courts', if the network drops and the client sends the request again, the user loses nothing because it is read-only. However, for 'POST /bookings', sending the request again could create a duplicate booking for the same court and time slot. Therefore, 'POST /bookings' needs an idempotency key so that retries of the same booking intention can be made harmless.


## Self-check Answers

## 1. Somebody suggests 'POST /v1/orders/{id}/markReady' . Give two reasons to reject it, and write the address you would use instead.
One reason I would reject this because 'markReady' is a verb, whilst the URI should represent a noun or resource. It also hides a state transition inside an action-style URI, which makes the API less consistent and harder for clients to understand. So, what i would do instead is use a noun or resource based address like 'POST /v1/orders/{id}/readiness' to represent the state transition

## 2.  A client sends 'PUT /v1/menu-items/itm_3Bn' with the complete item. It times out, so the client sends it again. Is that read-only? Is it safe to repeat? Answer the two questions separately — they are different properties, so one sentence covering both is wrong even if you reach the right conclusion.
What the client sends isn't only read-only because 'PUT' changes or replaces the menu item resouce. Moreover, it's safe to repeat because 'PUT' is idempotent, it sends the same complete representation again if the result were to be in the same final state as sending it once. Therefore, a timeout followed by the same 'PUT' request shouldn't create another menu item or cause the same change to happen multiple times.

## 3. A shop has sold out of an item. Which status code, and which of the three kinds of failure from Section 2.6? What is wrong with using 500? What is wrong with using 200?
For a sold out item, the status code I would use is '409 conflict' because it is a domain or business outcome; the item's current state conflict with the request. It shouldn't be '500' because the server hasn't failed internally; the request was understood but coudln't be fulfilled under the current business state. Moreover, using '200' would also be wrong since it makes a failed or refused operation look successful to the client/user.

## 4.  Your dangerous operation. Which operation in your practice system must never happen twice?
The dangerous opertaion in my practice system that shouldn't happen twice is 'POST /bookings' because it creates a badminton court booking. In the case of if the network going offline whilst a booking is being created and the user sends the request again, then the same booking could be created twice. This could cause duplicate bookings for the same court and time slot. Therefore, the 'createBooking' operation requires the 'Idempotency-Key' header, which is decalred as 'required: true' under the operation's parameters.

Moreover, in my system the user generates the 'Idempotency-Key' when they confirm the booking, and the same key is reused unchanged for every retry of that same booking intention. The key is retained for 24 hours, and reusing the same key with a different request body returns '409 idempotency-key-reuse'. My mock test also showed that omitting the required 'idempotency-key' results in '422 unprocessable entity', which confirms that the requirement is part of the written contract.

## 5. One thing you are unsure about. Genuinely unsure, not for show. Write it down.
As for what I'm still unsure about, it's how idempotency should be implemented in the actual service, especially how the server stores an idempotency-key, and the original response so that a retry can return the same booking instead of creating another one. I understand how to specify that requirement in OpenAPI, but I'm still not sure how the implementaton should handle concurrent requests using the same key.
