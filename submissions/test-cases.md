# Test Cases — STQA Group 31

| Info | |
|---|---|
| **Group** | Group 31 |
| **Date** | 23/05/2026 |
| **System Under Test** | https://stqa.rbc.vn |
| **Reference** | SRS v1.0 |

---

## Testing Technique Justification

This section explains **why** each testing technique was selected for each requirement, based on the nature of the input domain and business rules.

| REQ | Technique(s) Applied | Justification |
|-----|---------------------|---------------|
| REQ-01 | **EP** | Login has clearly separable valid/invalid input classes: email exists vs. not exists, password correct vs. wrong, fields empty vs. filled. EP efficiently covers each class with one representative — exhaustive testing is unnecessary. |
| REQ-02 | **EP** | Role-based access control has two distinct user classes (Librarian vs. Member), each with different view permissions. EP maps naturally to these partitions. No boundary values to analyse. |
| REQ-03 | **EP** | Search inputs partition into: keyword matches a title, keyword matches an author, keyword has no match, uppercase vs. lowercase. EP covers all meaningful classes. Filter by genre adds another partition axis. |
| REQ-04 | **EP + BVA + Decision Table** | EP covers book/member status classes. **BVA** is applied to the numeric borrow limit (0, 1, 2, 3 = at limit, 4 = over limit) because bugs most often occur at boundary values. **Decision Table** captures all combinations of three independent conditions (book available, borrow count < 3, account active), ensuring no rule combination is missed. |
| REQ-05 | **EP** | Return scenarios partition into: normal return (on time) and overdue return. EP covers both classes. No numeric boundaries to test. |
| REQ-06 | **EP** | Overdue detection separates records into: past due date (overdue) vs. within due date (not overdue). Two partitions, straightforward EP application. |
| REQ-07 | **EP + BVA** | Member addition partitions into: valid email format, invalid format (missing TLD), duplicate email, valid/invalid phone. **BVA** is applied to email format validation — testing the exact boundary between a valid format (`user@domain.com`) and an invalid one (`user@domain`) reveals whether the regex is permissive at the edge case. |
| REQ-08 | **EP** | Borrow record lookup partitions by role: Librarian sees all records vs. Member sees only own records. Additional partition: search by member ID (Librarian only). EP captures all access-control classes. |

**Summary of technique selection rationale:**
- **EP** is applied universally because every input domain has valid/invalid partitions and testing one value per partition is sufficient.
- **BVA** is applied to REQ-04 (numeric borrow limit) and REQ-07 (email format boundary) because defects are statistically more likely at boundaries than in the middle of a partition.
- **Decision Table** is applied exclusively to REQ-04 because it has three independent boolean conditions whose combinations produce different outcomes — a table ensures all 8 rule combinations are considered and no interaction is overlooked.

---

## Step 1: Input Domain Modeling (IDM)

### IDM — Login (REQ-01)

| Characteristic | Partition (Block) | Representative Value | Expected Result |
|---|---|---|---|
| Email exists in DB? | Yes | `librarian@library.com` | Login successful |
| | No | `noone@email.com` | "Member not found" |
| Password correct? | Correct | `admin123` | Login successful |
| | Wrong | `wrongpass` | "Incorrect password." |
| Input field empty? | Not empty | (valid value) | Normal processing |
| | Empty | `""` | "Please enter email and password" |

### IDM — View Book List (REQ-02)

| Characteristic | Partition (Block) | Representative Value | Expected Result |
|---|---|---|---|
| User role | Librarian | librarian@library.com | Can view all books |
| | Member | ba.nguyen@email.com | Can view all books |
| Book status | Available | BOOK001 | Shows "Available" |
| | Borrowed | BOOK003 | Shows "Borrowed" |
| | Lost | BOOK007 | Shows "Lost" |

### IDM — Search & Filter Books (REQ-03)

| Characteristic | Partition (Block) | Representative Value | Expected Result |
|---|---|---|---|
| Keyword exists? | Yes (book title) | `"Flutter"` | Displays books containing "Flutter" |
| | Yes (author name) | `"Nguyen Minh Duc"` | Displays books by that author |
| | No | `"XYZ999NOTEXIST"` | "No books found." |
| Case-sensitive? | Lowercase | `"flutter"` | Same result as "Flutter" |
| | Uppercase | `"FLUTTER"` | Same result as "Flutter" |
| Filter by genre | Existing genre | `"Economics"` | Displays Economics books only |
| | Non-existing genre | `"Physics"` | Empty list |

### IDM — Borrow Book (REQ-04)

| Characteristic | Partition (Block) | Representative Value | Expected Result |
|---|---|---|---|
| Book status | Available | BOOK001 | Borrowing allowed |
| | Borrowed | BOOK003 | Not allowed (button disabled) |
| Member status | Active | MEM003 | Borrowing allowed |
| | Suspended | MEM004 | Rejected with error message |
| | Expired | MEM005 | Rejected with error message |
| Books currently borrowed (BVA) | 0 books (lower boundary) | MEM003 initially | Borrowing allowed |
| | 1 book | MEM003 after 1 borrow | Borrowing allowed |
| | 2 books | MEM003 after 2 borrows | Borrowing allowed |
| | 3 books (at limit) | MEM003 after 3 borrows | Borrowing allowed |
| | 4 books (over limit) | MEM003 tries to borrow again | Rejected: "Limit of 3 books exceeded" |

### IDM — Return Book (REQ-05)

| Characteristic | Partition (Block) | Representative Value | Expected Result |
|---|---|---|---|
| Book currently borrowed? | Yes | BR001 (MEM002, BOOK003) | Return allowed |
| Borrow record status | Within due date | BR005 (due 15/12/2024) | Return successful |
| | Overdue | BR001 (due 15/09/2024) | Return successful + overdue warning shown |

### IDM — Overdue Handling (REQ-06)

| Characteristic | Partition (Block) | Representative Value | Expected Result |
|---|---|---|---|
| Borrow record overdue? | Yes (dueDate < today) | BR001 (due 15/09/2024) | Status changes to "Overdue" |
| | No | BR005 (due 15/12/2024) | Remains "Borrowed" |
| Who performs action | Librarian | librarian@library.com | Can click "Check Overdue" button |

### IDM — Member Management (REQ-07)

| Characteristic | Partition (Block) | Representative Value | Expected Result |
|---|---|---|---|
| Email valid? | Valid (has @ and .) | `newuser@example.com` | Added successfully |
| | Invalid (missing dot in domain) | `user@domain` | "Invalid email" error |
| | Invalid (missing @) | `userdomain.com` | Error message shown |
| Email already exists? | Not yet | `newuser999@test.com` | Added successfully |
| | Already exists | `ba.nguyen@email.com` | Error message shown |
| Who performs action | Librarian | librarian@library.com | Can add member |
| | Member | dam.tran@email.com | No add-member access |

### IDM — Borrow Record Lookup (REQ-08)

| Characteristic | Partition (Block) | Representative Value | Expected Result |
|---|---|---|---|
| User role | Librarian | librarian@library.com | Views all borrow records of all members |
| | Member | ba.nguyen@email.com | Views only own records |
| Search by member ID | Exists | MEM002 | Displays records of MEM002 |
| | Not exists | MEM999 | Empty list or "not found" message |

---

## Step 2: Decision Table — Borrow Book (REQ-04) — Bonus B2

| Condition / Action | DT-1 | DT-2 | DT-3 | DT-4 | DT-5 | DT-6 | DT-7 | DT-8 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Conditions** | | | | | | | | |
| Book status is "Available"? | T | T | T | T | F | F | F | F |
| Member has < 3 books borrowed? | T | T | F | F | T | T | F | F |
| Member status is "Active"? | T | F | T | F | T | F | T | F |
| **Actions** | | | | | | | | |
| Allow borrowing | **✅** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Error: Account suspended/expired | | **✅** | | **✅** | | **✅** | | **✅** |
| Error: Borrow limit of 3 reached | | | **✅** | **✅** | | | **✅** | **✅** |
| Error: Book is already borrowed | | | | | **✅** | **✅** | **✅** | **✅** |

> Note: When multiple conditions fail, the system prioritizes the first detected error.

---

## Step 3: Test Cases

### REQ-01: Login

| TC ID | Test Objective | Preconditions | Steps | Input Data | Expected Result | REQ | Technique |
|-------|---------------|---------------|-------|------------|-----------------|-----|-----------|
| TC-01 | Login with both fields empty | Login page open, data reset | 1. Leave Email and Password blank. 2. Click "Login". | Email: `""`, Password: `""` | Error message: "Please enter email and password". Page does not navigate. | REQ-01 | EP |
| TC-02 | Login with non-existent email | Login page open | 1. Enter email not in system. 2. Enter any password. 3. Click "Login". | Email: `noone@email.com`, PW: `anypass123` | Message: "Member not found.". Page does not navigate. | REQ-01 | EP |
| TC-03 | Login with correct email but wrong password | Login page open | 1. Enter valid email. 2. Enter wrong password. 3. Click "Login". | Email: `librarian@library.com`, PW: `wrongpass` | Message: "Incorrect password.". Page does not navigate. | REQ-01 | EP |
| TC-04 | Successful login as Librarian | Login page open | 1. Enter librarian email. 2. Enter correct password. 3. Click "Login". | Email: `librarian@library.com`, PW: `admin123` | Redirects to home page. AppBar shows username and role "(Librarian)". | REQ-01 | EP |
| TC-28 | Login with only password field empty | Login page open | 1. Enter valid email. 2. Leave Password blank. 3. Click "Login". | Email: `librarian@library.com`, PW: `""` | Error message: "Please enter email and password". Page does not navigate. | REQ-01 | EP |
| TC-29 | Successful login as Member | Login page open | 1. Enter member email. 2. Enter correct password. 3. Click "Login". | Email: `dam.tran@email.com`, PW: `password123` | Redirects to home page. AppBar shows username and role "(Member)". Navigation shows only "Books" and "Borrow / Return" tabs. | REQ-01 | EP |

### REQ-02: View Book List

| TC ID | Test Objective | Preconditions | Steps | Input Data | Expected Result | REQ | Technique |
|-------|---------------|---------------|-------|------------|-----------------|-----|-----------|
| TC-05 | Librarian views complete book list with all details | Logged in as Librarian | 1. Observe "Books" tab. | — | List displays at least 18 books. Each book shows: title, author, genre, year, status. BOOK003 shows "Borrowed". | REQ-02 | EP |
| TC-06 | Member does not see "Members" tab | Logged in as Member `dam.tran@email.com / password123` | 1. Observe bottom navigation bar. | — | Only 2 tabs shown: "Books" and "Borrow / Return". No "Members" tab visible. | REQ-02 | EP |
| TC-30 | Book with "Lost" status is shown in list | Logged in as Librarian | 1. Go to "Books" tab. 2. Locate BOOK007. 3. Observe displayed status. | Book: BOOK007 | BOOK007 appears in the list with status "Lost" (or equivalent Vietnamese label). | REQ-02 | EP |

### REQ-03: Search & Filter

| TC ID | Test Objective | Preconditions | Steps | Input Data | Expected Result | REQ | Technique |
|-------|---------------|---------------|-------|------------|-----------------|-----|-----------|
| TC-07 | Search is case-insensitive | Logged in, on "Books" tab | 1. Enter lowercase keyword in search box. 2. Observe results. | Keyword: `flutter` | Displays "Flutter Programming Basics" (BOOK001). Same result as searching "Flutter". | REQ-03 | EP |
| TC-08 | Search for non-existent keyword | Logged in, on "Books" tab | 1. Enter keyword not in DB. 2. Observe results. | Keyword: `XYZ999NOTEXIST` | Message: "No books found.". List is empty. | REQ-03 | EP |
| TC-09 | Filter books by genre | Logged in, on "Books" tab | 1. Enter genre in filter box. 2. Observe results. | Genre: `Kinh tế` | Only Economics books displayed (BOOK007, BOOK014, BOOK015). | REQ-03 | EP |
| TC-31 | Search by author name — returns matching books | Logged in, on "Books" tab | 1. Enter an author name that exists in the DB. 2. Observe results. | Keyword: `Nguyen Minh Duc` | All books whose author name contains "Nguyen Minh Duc" are displayed. No unrelated books shown. | REQ-03 | EP |
| TC-32 | Partial keyword search returns all matching results | Logged in, on "Books" tab | 1. Enter a partial keyword that matches multiple titles. 2. Observe results. | Keyword: `Python` | All books with "Python" in title or author are displayed (at least 2 results if available). Search is not limited to exact match. | REQ-03 | EP |
| TC-39 | Category filter works correctly in English UI mode | Logged in, UI switched to EN mode | 1. Click "EN" language toggle. 2. Click the category filter field. 3. Type the English category name "Technology". 4. Observe results. | Category: `Technology` (EN mode active) | Books in the "Technology" / "Công nghê" category are displayed. Filter should work regardless of UI language. | REQ-03 | EP |

### REQ-04: Borrow Book

| TC ID | Test Objective | Preconditions | Steps | Input Data | Expected Result | REQ | Technique |
|-------|---------------|---------------|-------|------------|-----------------|-----|-----------|
| TC-10 | Borrow available book — happy path | Logged in as MEM003 (`dam.tran@email.com`), 0 books borrowed | 1. On "Books" tab, click "+" next to BOOK001. 2. Confirm in dialog. | Book: BOOK001, Member: MEM003 | Toast: "Book borrowed successfully!". BOOK001 status changes to "Borrowed". | REQ-04 | EP, DT-1 |
| TC-11 | Borrow 2nd book (BVA — 1 < limit 3) | Logged in as MEM003, currently borrowing 1 book | 1. Click "+" next to BOOK002. 2. Confirm in dialog. | Book: BOOK002, Member: MEM003 | Toast: "Book borrowed successfully!". BOOK002 status changes to "Borrowed". | REQ-04 | BVA |
| TC-12 | Borrow 3rd book (BVA — exactly at limit 3) | Logged in as MEM003, currently borrowing 2 books | 1. Click "+" next to BOOK005. 2. Confirm in dialog. | Book: BOOK005, Member: MEM003 | Toast: "Book borrowed successfully!". BOOK005 status changes to "Borrowed". | REQ-04 | BVA |
| TC-13 | Borrow 4th book — exceeds limit (BVA — 4 > limit 3) | Logged in as MEM003, currently borrowing 3 books | 1. Click "+" next to BOOK008. 2. Confirm in dialog. | Book: BOOK008, Member: MEM003 | System rejects. Error message: "Borrow limit of 3 books reached" or equivalent. BOOK008 remains "Available". | REQ-04 | BVA, DT-3 |
| TC-14 | Borrow rejected for suspended account | Login as MEM004 (`cu.le@email.com / password123`) | 1. Login as MEM004. 2. Click "+" next to any available book. 3. Confirm borrow. | Member: MEM004 (Suspended), Book: BOOK001 | System rejects borrow. Error message describes suspension reason. | REQ-04 | EP, DT-2 |
| TC-15 | Borrow rejected for expired account | Login as MEM005 (`binh.pham@email.com / password123`) | 1. Login as MEM005. 2. Click "+" next to any available book. 3. Confirm borrow. | Member: MEM005 (Expired), Book: BOOK001 | System rejects borrow. Error message describes expiration reason (different from suspended). | REQ-04 | EP, DT-2 |
| TC-33 | Borrow rejected when book is already "Borrowed" — button disabled | Logged in as MEM003, BOOK003 status is "Borrowed" | 1. Go to "Books" tab. 2. Locate BOOK003 (status Borrowed). 3. Observe the "+" borrow button. | Book: BOOK003 (Borrowed), Member: MEM003 | The "+" borrow button is disabled or hidden for BOOK003. No borrow dialog appears. BOOK003 remains "Borrowed". | REQ-04 | EP, DT-5 |

### REQ-05: Return Book

| TC ID | Test Objective | Preconditions | Steps | Input Data | Expected Result | REQ | Technique |
|-------|---------------|---------------|-------|------------|-----------------|-----|-----------|
| TC-16 | Return book successfully | Logged in as Librarian, BR003 status is "Borrowed" | 1. Go to "Borrow / Return" tab. 2. Find record BR003. 3. Click "Return Book". | Record: BR003 (BOOK013, MEM006) | Toast: "Book returned successfully.". BR003 status changes to "Returned". BOOK013 changes to "Available". | REQ-05 | EP |
| TC-17 | Return overdue book — check warning message | Logged in as Librarian, BR001 marked as overdue | 1. Click "Check Overdue Books". 2. Find BR001 (status Overdue). 3. Click "Return Book". | Record: BR001 (overdue since 15/09/2024) | Message includes overdue warning (in addition to "returned successfully"). BR001 status changes to "Returned". | REQ-05 | EP |

### REQ-06: Overdue Handling

| TC ID | Test Objective | Preconditions | Steps | Input Data | Expected Result | REQ | Technique |
|-------|---------------|---------------|-------|------------|-----------------|-----|-----------|
| TC-18 | Librarian runs overdue check — records update status | Logged in as Librarian, BR001 (due 15/09/2024) is "Borrowed" | 1. Go to "Borrow / Return" tab. 2. Click "Check Overdue Books" button. | — | System scans all records. BR001 and BR003 (due 15/10/2024) change to "Overdue". Count of overdue records is displayed. | REQ-06 | EP |
| TC-19 | Member views own overdue record | Logged in as MEM002 (`ba.nguyen@email.com`), BR001 is overdue | 1. Login as `ba.nguyen@email.com / password123`. 2. Go to "Borrow / Return" tab. | — | Displays BR001 with status "Overdue". Records from other members are not shown. | REQ-06 | EP |

### REQ-07: Member Management

| TC ID | Test Objective | Preconditions | Steps | Input Data | Expected Result | REQ | Technique |
|-------|---------------|---------------|-------|------------|-----------------|-----|-----------|
| TC-20 | Add new member with valid data | Logged in as Librarian | 1. Click "Add Member" icon on AppBar. 2. Enter valid name, email, phone. 3. Click "Add Member". | Name: `Le Van Test`, Email: `levantest@example.com`, Phone: `0912000001` | Success message with new member ID (e.g. MEM008). New member appears in list. | REQ-07 | EP |
| TC-21 | Add member with invalid email — missing dot in domain | Logged in as Librarian, Add Member form open | 1. Enter email in format `user@domain` (no dot in domain). 2. Fill other fields with valid data. 3. Click "Add Member". | Email: `user@domain`, Name: `Test User`, Phone: `0900000002` | System rejects. Message: "Invalid email" or equivalent. No new member created. | REQ-07 | EP, BVA |
| TC-22 | Add member with already-existing email | Logged in as Librarian | 1. Enter an email already in the system. 2. Fill other fields with valid data. 3. Click "Add Member". | Email: `ba.nguyen@email.com` (existing — MEM002) | System rejects. Error message clearly states email already exists (NOT "invalid email"). | REQ-07 | EP |
| TC-23 | Regular member cannot access member management | Logged in as Member | 1. Observe UI after login with member account. | Member: `dam.tran@email.com / password123` | No "Add Member" icon on AppBar. No "Members" tab shown. | REQ-07 | EP |
| TC-34 | Add member with email missing "@" symbol — rejected | Logged in as Librarian, Add Member form open | 1. Enter email without "@" symbol. 2. Fill other fields with valid data. 3. Click "Add Member". | Email: `userdomain.com`, Name: `Test NoAt`, Phone: `0900000010` | System rejects. Validation error displayed (e.g. "Invalid email"). No new member is created. | REQ-07 | EP, BVA |
| TC-36 | Add member with purely non-numeric phone — server-side rejection check | Logged in as Librarian, Add Member form open | 1. Enter valid name and email. 2. Enter an alphabetic phone number. 3. Click "Add Member". | Name: `Le Van Test`, Email: `levantest2@example.com`, Phone: `abcdefghij` | System rejects. Error: "Invalid phone number format." No new member created. | REQ-07 | EP |
| TC-37 | Add member with empty name field — check error specificity | Logged in as Librarian, Add Member form open | 1. Leave name field empty. 2. Enter valid email and phone. 3. Click "Add Member". | Name: *(empty)*, Email: `validuser@example.com`, Phone: `0912000002` | System rejects. Error specifically identifies name field (e.g., "Name is required."). NOT "Email không hợp lệ.". | REQ-07 | EP |
| TC-38 | Add member with empty phone field — check error specificity | Logged in as Librarian, Add Member form open | 1. Enter valid name and email. 2. Leave phone field empty. 3. Click "Add Member". | Name: `Test Empty Phone`, Email: `emptyphonetest@example.com`, Phone: *(empty)* | System rejects. Error specifically identifies phone field (e.g., "Phone number is required."). NOT "Email không hợp lệ.". | REQ-07 | EP |

### REQ-08: Borrow Record Lookup

| TC ID | Test Objective | Preconditions | Steps | Input Data | Expected Result | REQ | Technique |
|-------|---------------|---------------|-------|------------|-----------------|-----|-----------|
| TC-24 | Librarian views all borrow records | Logged in as Librarian | 1. Go to "Borrow / Return" tab → "All Records" sub-tab. | — | All borrow records from all members displayed (BR001–BR005). Each record shows: record ID, book, member, borrow date, due date, status. | REQ-08 | EP |
| TC-25 | Librarian searches records by member ID | Logged in as Librarian | 1. Go to "Borrow / Return" tab → "Search Records" sub-tab. 2. Enter member ID. 3. Click "Search". | Member ID: `MEM002` | Displays MEM002's records: BR001 (Borrowed/Overdue) and BR004 (Returned). Records of other members not shown. | REQ-08 | EP |
| TC-26 | Member sees only own borrow records | Logged in as MEM002 (`ba.nguyen@email.com`) | 1. Go to "Borrow / Return" tab. | — | Only MEM002's records shown (BR001, BR004). MEM003's record (BR002) and MEM006's record (BR003) not visible. | REQ-08 | EP |
| TC-27 | Member cannot search by other member IDs | Logged in as MEM002 | 1. Go to "Borrow / Return" tab → check if member-ID search box exists. | — | No member-ID search input available. Member sees only own records automatically. | REQ-08 | EP |
| TC-35 | Librarian searches records by non-existent member ID — no results | Logged in as Librarian | 1. Go to "Borrow / Return" tab → "Search Records" sub-tab. 2. Enter member ID that does not exist. 3. Click "Search". | Member ID: `MEM999` | System returns empty list or displays "No records found." message. No error crash. | REQ-08 | EP |

---

## Summary

| Feature Group | TCs | TC IDs | REQ Coverage | Techniques Applied |
|--------------|-----|--------|-------------|-------------------|
| Login | 6 | TC-01 to TC-04, TC-28, TC-29 | REQ-01 | EP |
| View Book List | 3 | TC-05, TC-06, TC-30 | REQ-02 | EP |
| Search & Filter | 6 | TC-07 to TC-09, TC-31, TC-32, TC-39 | REQ-03 | EP |
| Borrow Book | 7 | TC-10 to TC-15, TC-33 | REQ-04 | EP, BVA, Decision Table |
| Return Book | 2 | TC-16, TC-17 | REQ-05 | EP |
| Overdue Handling | 2 | TC-18, TC-19 | REQ-06 | EP |
| Member Management | 8 | TC-20 to TC-23, TC-34, TC-36, TC-37, TC-38 | REQ-07 | EP, BVA |
| Borrow Record Lookup | 5 | TC-24 to TC-27, TC-35 | REQ-08 | EP |
| **Total** | **39** | **TC-01 – TC-39** | **8/8 REQ** | **EP + BVA + Decision Table** |
