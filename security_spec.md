# Security Specification - Bunyoro Rides

## Data Invariants
1. **User Identity**: Users can only create and update their own profiles.
2. **Role Integrity**: A user's `role` cannot be changed after initial selection.
3. **Ride Creation**: Only users with the `client` role can create ride requests.
4. **Ride Acceptance**: Only users with the `rider` role can accept.
5. **Phone Verification**: Rules check for `request.auth.token.phone_number`.
6. **State Transitions**: Ride status must follow a strict sequence: `requested` -> `accepted` -> `picked_up` -> `completed`.
7. **Immutable Fields**: `clientId` and `phone` are immutable after creation.

## The "Dirty Dozen" Payloads (Denial Expected)
1. **Identity Spoofing**: Attempt to update another user's profile.
2. **Role Hijacking**: Client attempts to update their role to `admin` or `rider`.
3. **Ghost Ride**: Unauthorized user attempts to create a ride request.
4. **Snatching**: Rider A attempts to modify a ride already accepted by Rider B.
5. **Status Skipping**: Client attempts to move status from `requested` directly to `completed`.
6. **Phantom Accept**: Rider attempts to accept a ride that doesn't exist (handled by Firestore logic, but rule-checked).
7. **Price Tampering**: Client attempts to lower the price of an accepted ride.
8. **ID Poisoning**: Use a 2KB string as a `userId` or `rideId`.
9. **Shadow Data**: Adding `isAdmin: true` to a user profile payload.
10. **Time Warp**: Client provides an old `createdAt` timestamp.
11. **Orphan Ride**: Creating a ride with a `clientId` that doesn't match the authenticated user.
12. **Double Accept**: Rider B attempts to update status to `accepted` when it's already `accepted` by Rider A.

## Implementation Details
The rules will use `affectedKeys()` to ensure only permitted fields are updated during specific actions (e.g., Accepting a ride only updates `status`, `riderId`, and `updatedAt`).
