**Hosted application:** [https://cisc472-assignment5.onrender.com/](https://cisc472-assignment5.onrender.com/)

# Broken Access Control

**Location:** `market/views.py`, `listing_edit`, `listing_sold`, and `listing_delete`

**Trigger:** A user who is logged in can change the listing ID in the URL or POST request to edit, mark as sold, or remove another user's listing. The views check that the user is authenticated, but they do not check that the user owns the listing.

*Example:* Alex owns the listing at `/listings/4/`. While signed in as Jordan, an attacker visits `/listings/4/edit/`, changes the title, and submits the form. Jordan can modify Alex's listing even though Jordan is not the owner. Similarly, while signed in as Jordan, submitting a POST request to `/listings/4/sold/` changes Alex's listing status to sold.


**The Fix**:
Check that the listing belongs to the logged-in user before changing it:

```python
listing = get_object_or_404(Listing, pk=pk, owner=request.user)
```

Use this check in the edit, sold, and delete views. If the user is not the owner, the listing is not changed.

**how a listing id becomes a row:**
When a user submits the new-listing form, `listing_new` creates a `Listing` object, sets its owner to the logged-in user, and calls `listing.save()`. Django inserts it into the database and automatically gives it an integer primary-key ID. That ID is used in URLs such as `/listings/4/` to find the row.

**which pages a logged-out visitor can open**
Logged-out visitors can open the home board, individual listing pages, the login page, and the registration page. They cannot create listings, view their own page, message sellers, edit listings, view their profile, or open the department desk until they sign in.

**what the profile form writes back to the user:**
The profile form updates the logged-in Django user’s first name, last name, email address, and `is_staff` value. The `is_staff` field is intended to control access to the department desk, but allowing users to edit it creates the privilege-escalation flaw described above.

**what the department desk is for, and who can open it:**
The department desk displays private department information, including the hold-closet location and reimbursement code. Only users whose account has `is_staff=True` can view that information. Other logged-in users can reach the page but receive an access-denied message.