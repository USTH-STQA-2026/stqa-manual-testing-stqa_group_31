# Test Execution Results — STQA Group 31

| Info | |
|---|---|
| **Group** | Group 31 |
| **Execution Date** | 23/05/2026 |
| **Tester** | Group 31 |
| **System Under Test** | https://stqa.rbc.vn |
| **Environment** | Chrome browser, Flutter Web (CanvasKit) |
| **Reference** | test-cases.md |

---

## Execution Summary

| Total TCs | PASS | FAIL | N/T |
|-----------|------|------|-----|
| 35 | 31 | 4 | 0 |

---

## Detailed Execution Results

### REQ-01: Login

| TC ID | Test Objective | Input Data | Expected Result | Actual Result | Status | Bug Ref | Notes |
|-------|---------------|------------|-----------------|---------------|--------|---------|-------|
| TC-01 | Login with both fields empty | Email: `""`, PW: `""` | Error: "Please enter email and password" | System displayed error message preventing login | **PASS** | — | Vietnamese message: "Vui lòng nhập email và mật khẩu" |
| TC-02 | Login with non-existent email | Email: `noone@email.com`, PW: `anypass123` | Message: "Member not found." | System displayed "Không tìm thấy thành viên." toast. Did not navigate. | **PASS** | — | |
| TC-03 | Login with wrong password | Email: `librarian@library.com`, PW: `wrongpass` | Message: "Incorrect password." | System displayed "Mật khẩu không đúng." toast. Did not navigate. | **PASS** | — | |
| TC-04 | Successful login as Librarian | Email: `librarian@library.com`, PW: `admin123` | Redirect to home, AppBar shows role | Redirected to home page. AppBar shows "Nguyễn Văn Thư (Thủ thư)". | **PASS** | — | |
| TC-28 | Login with only password field empty | Email: `librarian@library.com`, PW: `""` | Error: "Please enter email and password" | System displayed "Vui lòng nhập email và mật khẩu." toast. Page did not navigate. | **PASS** | — | Same validation as TC-01 triggered by empty password |
| TC-29 | Successful login as Member | Email: `dam.tran@email.com`, PW: `password123` | Redirect to home, AppBar shows "(Member)" role | Redirected to home page. AppBar shows "Trần Thị Đầm (Thành viên)". Only "Sách" and "Mượn / Trả" tabs shown. | **PASS** | — | |

### REQ-02: View Book List

| TC ID | Test Objective | Input Data | Expected Result | Actual Result | Status | Bug Ref | Notes |
|-------|---------------|------------|-----------------|---------------|--------|---------|-------|
| TC-05 | Librarian views complete book list | — | ≥18 books shown with full details | All 20 books displayed. Each shows title, author, genre, year, status. BOOK003 shows "Đang mượn" (Borrowed). | **PASS** | — | |
| TC-06 | Member does not see "Members" tab | Member: `dam.tran@email.com` | Only "Books" and "Borrow/Return" tabs | Only 2 tabs shown: "Sách" and "Mượn / Trả". No "Thành viên" (Members) tab visible. | **PASS** | — | |
| TC-30 | Book with "Lost" status is shown in list | — | BOOK007 appears with status "Lost" | BOOK007 "Kinh tế học vĩ mô" listed with status "Mất sách" (Lost). Status displayed correctly in book list. | **PASS** | — | |

### REQ-03: Search & Filter

| TC ID | Test Objective | Input Data | Expected Result | Actual Result | Status | Bug Ref | Notes |
|-------|---------------|------------|-----------------|---------------|--------|---------|-------|
| TC-07 | Case-insensitive search | Keyword: `flutter` | Same results as "Flutter" | BOOK001 "Lập trình Flutter cơ bản" displayed. Result matches uppercase search. | **PASS** | — | |
| TC-08 | Search non-existent keyword | Keyword: `XYZ999NOTEXIST` | "No books found." | Empty list shown with "Không tìm thấy sách nào." message. | **PASS** | — | |
| TC-09 | Filter by genre | Genre: `Kinh tế` | Only Economics books | Filtered to show only genre "Kinh tế" books (BOOK007, BOOK014, BOOK015). | **PASS** | — | |
| TC-31 | Search by author name | Keyword: `Nguyen Minh Duc` | Books by that author displayed | Books authored by "Nguyễn Minh Đức" displayed. No unrelated books shown. Search works on author field. | **PASS** | — | |
| TC-32 | Partial keyword search returns multiple results | Keyword: `Python` | All books with "Python" in title shown | Books containing "Python" in title displayed (BOOK002 "Lập trình Python nâng cao", BOOK010 "Python for Data Science"). List is not limited to exact match. | **PASS** | — | |

### REQ-04: Borrow Book

| TC ID | Test Objective | Input Data | Expected Result | Actual Result | Status | Bug Ref | Notes |
|-------|---------------|------------|-----------------|---------------|--------|---------|-------|
| TC-10 | Borrow available book — happy path | BOOK001, MEM003 | Success toast. Book status → "Borrowed" | Toast "Mượn sách thành công!" displayed. BOOK001 status changed to "Đang mượn". | **PASS** | — | |
| TC-11 | Borrow 2nd book (BVA: 1 book) | BOOK002, MEM003 | Success toast | Toast "Mượn sách thành công!" displayed. BOOK002 status changed to "Đang mượn". | **PASS** | — | |
| TC-12 | Borrow 3rd book (BVA: at limit 3) | BOOK005, MEM003 | Success toast | Toast "Mượn sách thành công!" displayed. BOOK005 status changed to "Đang mượn". | **PASS** | — | |
| TC-13 | Borrow 4th book (BVA: over limit) | BOOK008, MEM003 | System rejects. Error message. BOOK008 stays "Available" | **System accepted the 4th borrow.** Toast "Mượn sách thành công!" shown. BOOK008 changed to "Đang mượn". No error message. | **FAIL** | BUG-04 | Critical bug: borrow limit not enforced |
| TC-14 | Borrow by suspended account | MEM004, BOOK001 | System rejects with suspension message | MEM004 login succeeds but "+" borrow button is disabled/not shown for all books. Cannot initiate borrow. | **PASS** | — | UI prevents borrow action for suspended accounts |
| TC-15 | Borrow by expired account | MEM005, BOOK001 | System rejects with expiration message | MEM005 login succeeds but "+" borrow button is disabled/not shown for all books. Cannot initiate borrow. | **PASS** | — | UI prevents borrow action for expired accounts |
| TC-33 | Borrow button disabled for already-borrowed book | Book: BOOK003 (Borrowed), Member: MEM003 | "+" button disabled or hidden for BOOK003 | BOOK003 displayed with status "Đang mượn". The "+" borrow button is not rendered for this book. No borrow dialog can be triggered. | **PASS** | — | UI correctly blocks borrowing of unavailable books |

### REQ-05: Return Book

| TC ID | Test Objective | Input Data | Expected Result | Actual Result | Status | Bug Ref | Notes |
|-------|---------------|------------|-----------------|---------------|--------|---------|-------|
| TC-16 | Return book successfully | Record: BR003 (BOOK013, MEM006) | "Book returned successfully." BR003 → Returned. BOOK013 → Available | Toast "Trả sách thành công." shown. BR003 status changed to "Đã trả". BOOK013 changed to "Có sẵn". | **PASS** | — | |
| TC-17 | Return overdue book — warning check | Record: BR001 (overdue since 15/09/2024) | Toast includes overdue warning + success. BR001 → Returned | Toast showed only "Trả sách thành công." **No overdue warning displayed.** BR001 status changed to "Đã trả". | **FAIL** | BUG-01 | Missing overdue notification on return |

### REQ-06: Overdue Handling

| TC ID | Test Objective | Input Data | Expected Result | Actual Result | Status | Bug Ref | Notes |
|-------|---------------|------------|-----------------|---------------|--------|---------|-------|
| TC-18 | Overdue check updates record statuses | — | BR001 and BR003 change to "Overdue" | Clicked "Kiểm tra sách quá hạn". BR001 and BR002 status changed to "Quá hạn". Count of overdue records shown in toast. | **PASS** | — | |
| TC-19 | Member views own overdue record | MEM002 (`ba.nguyen@email.com`) | BR001 shown as "Overdue". No other records. | After login as MEM002 → "Mượn / Trả" tab shows BR001 with status "Quá hạn". No other member's records visible. | **PASS** | — | |

### REQ-07: Member Management

| TC ID | Test Objective | Input Data | Expected Result | Actual Result | Status | Bug Ref | Notes |
|-------|---------------|------------|-----------------|---------------|--------|---------|-------|
| TC-20 | Add new member with valid data | Name: `Le Van Test`, Email: `levantest@example.com`, Phone: `0912000001` | Success message with new member ID | Toast "Thêm thành viên thành công! Mã: MEM00X" displayed. New member appeared in list. | **PASS** | — | |
| TC-21 | Add member with invalid email (`user@domain`) | Email: `user@domain` | System rejects. "Invalid email" error | **System accepted the submission.** Toast "Thêm thành viên thành công! Mã: MEM007" displayed. Member created with invalid email format. | **FAIL** | BUG-02 | High severity: no server-side email format validation |
| TC-22 | Add member with duplicate email | Email: `ba.nguyen@email.com` (MEM002) | Reject. Error: "Email already exists" | System rejected the submission but displayed misleading message "Email không hợp lệ." ("Invalid email") instead of indicating the email already exists. | **FAIL** | BUG-03 | Wrong error message confuses users |
| TC-23 | Regular member cannot manage members | Member: `dam.tran@email.com` | No "Add Member" icon. No "Members" tab. | After member login: no add-member icon in AppBar, no "Thành viên" tab in navigation. | **PASS** | — | |
| TC-34 | Add member with email missing "@" symbol | Email: `userdomain.com`, Name: `Test NoAt`, Phone: `0900000010` | System rejects with validation error | System displayed validation error "Email không hợp lệ." and rejected the submission. No new member created. The missing "@" was detected by client-side validation. | **PASS** | — | Basic "@" check works; dot-in-domain check does not (BUG-02) |

### REQ-08: Borrow Record Lookup

| TC ID | Test Objective | Input Data | Expected Result | Actual Result | Status | Bug Ref | Notes |
|-------|---------------|------------|-----------------|---------------|--------|---------|-------|
| TC-24 | Librarian views all borrow records | — | All records BR001–BR005 displayed | All borrow records visible in "Mượn / Trả" tab: BR001, BR002, BR003, BR004, BR005 with full details. | **PASS** | — | |
| TC-25 | Librarian searches by member ID | Member ID: `MEM002` | BR001 and BR004 displayed | Entered MEM002 in search. BR001 (Quá hạn) and BR004 (Đã trả) returned. Correct. | **PASS** | — | |
| TC-26 | Member sees only own records | MEM002 (`ba.nguyen@email.com`) | Only BR001 and BR004 visible | After login as MEM002: only BR001 and BR004 shown. BR002, BR003, BR005 not visible. | **PASS** | — | |
| TC-27 | Member cannot search by member ID | MEM002 | No member-ID search box | "Mượn / Trả" tab for member shows no search input. Records filtered automatically to logged-in member only. | **PASS** | — | |
| TC-35 | Search records by non-existent member ID | Member ID: `MEM999` | Empty list or "No records found." | Entered MEM999 in member-ID search. System returned empty list with "Không tìm thấy bản ghi." message. No error crash. | **PASS** | — | |

---

## Defect Summary

| Bug ID | Severity | TC | Description |
|--------|----------|----|-------------|
| BUG-01 | Medium | TC-17 | No overdue warning displayed when returning an overdue book |
| BUG-02 | High | TC-21 | Invalid email format `user@domain` accepted when adding a member |
| BUG-03 | Low | TC-22 | Duplicate email shows misleading "Invalid email" message instead of "Email already exists" |
| BUG-04 | Critical | TC-13 | System allows borrowing beyond the 3-book limit per member |

---

## Test Coverage

| REQ | TCs | PASS | FAIL | Coverage |
|-----|-----|------|------|----------|
| REQ-01 | 6 | 6 | 0 | ✅ Full |
| REQ-02 | 3 | 3 | 0 | ✅ Full |
| REQ-03 | 5 | 5 | 0 | ✅ Full |
| REQ-04 | 7 | 6 | 1 (TC-13) | ⚠️ Bug found |
| REQ-05 | 2 | 1 | 1 (TC-17) | ⚠️ Bug found |
| REQ-06 | 2 | 2 | 0 | ✅ Full |
| REQ-07 | 5 | 3 | 2 (TC-21, TC-22) | ⚠️ Bugs found |
| REQ-08 | 5 | 5 | 0 | ✅ Full |
| **Total** | **35** | **31** | **4** | **8/8 REQ tested** |
