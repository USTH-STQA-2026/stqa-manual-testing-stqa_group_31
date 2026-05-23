# Summary Report — STQA Group 31

| Info | |
|---|---|
| **Group** | Group 31 |
| **Assignment** | A1 — Manual Black-Box Testing |
| **System Under Test** | https://stqa.rbc.vn (Library Management System) |
| **Testing Type** | Manual Black-Box Testing |
| **Date** | 23/05/2026 |
| **Reference** | SRS v1.0, test-cases.md, test-execution.md, bug-reports.md |

---

## 1. Overview

This report summarizes the results of manual black-box testing performed on the Library Management System (LMS) at https://stqa.rbc.vn. The testing covered all 8 functional requirements defined in the SRS (REQ-01 through REQ-08) using Equivalence Partitioning (EP), Boundary Value Analysis (BVA), and Decision Table techniques.

A total of **27 test cases** were designed and executed. **4 defects** were identified, including one critical bug that violates a core business rule.

---

## 2. Testing Scope

| Requirement | Description | TCs Designed | TCs Executed |
|-------------|-------------|:------------:|:------------:|
| REQ-01 | User Login | 4 | 4 |
| REQ-02 | View Book List | 2 | 2 |
| REQ-03 | Search & Filter Books | 3 | 3 |
| REQ-04 | Borrow Book | 6 | 6 |
| REQ-05 | Return Book | 2 | 2 |
| REQ-06 | Overdue Handling | 2 | 2 |
| REQ-07 | Member Management | 4 | 4 |
| REQ-08 | Borrow Record Lookup | 4 | 4 |
| **Total** | | **27** | **27** |

---

## 3. Test Results Summary

| Metric | Value |
|--------|-------|
| Total Test Cases | 27 |
| Passed | 23 |
| Failed | 4 |
| Pass Rate | 85.2% |
| Requirements Covered | 8 / 8 (100%) |
| Defects Found | 4 |
| Critical Defects | 1 (BUG-04) |
| High Defects | 1 (BUG-02) |
| Medium Defects | 1 (BUG-01) |
| Low Defects | 1 (BUG-03) |

### Pass/Fail by Requirement

| REQ | Pass | Fail | Result |
|-----|:----:|:----:|--------|
| REQ-01 | 4 | 0 | ✅ All Pass |
| REQ-02 | 2 | 0 | ✅ All Pass |
| REQ-03 | 3 | 0 | ✅ All Pass |
| REQ-04 | 5 | 1 | ⚠️ BUG-04 (Critical) |
| REQ-05 | 1 | 1 | ⚠️ BUG-01 (Medium) |
| REQ-06 | 2 | 0 | ✅ All Pass |
| REQ-07 | 2 | 2 | ⚠️ BUG-02 (High) + BUG-03 (Low) |
| REQ-08 | 4 | 0 | ✅ All Pass |

---

## 4. Defects Found

| Bug ID | Title | Severity | REQ | TC |
|--------|-------|----------|-----|----|
| BUG-04 | System allows borrowing beyond the 3-book limit per member | **Critical** | REQ-04 | TC-13 |
| BUG-02 | Invalid email format `user@domain` accepted when adding a member | **High** | REQ-07 | TC-21 |
| BUG-01 | No overdue warning displayed when returning an overdue book | **Medium** | REQ-05 | TC-17 |
| BUG-03 | Misleading "Invalid email" message shown for duplicate email | **Low** | REQ-07 | TC-22 |

---

## 5. Testing Techniques Applied

### Equivalence Partitioning (EP)

Applied to all 8 requirements. Each input domain was partitioned into valid and invalid equivalence classes, with one representative test case per class. This technique ensured broad coverage across diverse input conditions without exhaustive testing.

**Example** — REQ-01 Login: partitioned into (email exists / not exists), (password correct / wrong), (fields empty / not empty) → 4 test cases covering all major partitions.

### Boundary Value Analysis (BVA)

Applied to REQ-04 (Borrow Book) for the 3-book borrow limit. Boundary values tested:
- 0 books (lower boundary) → TC-10
- 1 book (within range) → TC-11
- 2 books (within range) → TC-11
- 3 books (at upper boundary / limit) → TC-12
- 4 books (over boundary) → TC-13 (**BUG-04 discovered here**)

BVA was also applied to REQ-07 email validation (testing the boundary between valid and invalid email formats).

### Decision Table (REQ-04 — Bonus B2)

A decision table with 3 conditions and 8 rule combinations was constructed for the borrow-book feature:
- Condition 1: Book is "Available"
- Condition 2: Member has < 3 books borrowed
- Condition 3: Member status is "Active"

Only rule DT-1 (all conditions true) should allow borrowing. All other combinations should produce specific error messages. The decision table revealed that **BUG-04** causes DT-3 (over-limit) to behave identically to DT-1 (allowed).

---

## 6. Quality Assessment

### System Strengths

- **Authentication** (REQ-01) is robust — empty input, non-existent email, and wrong password are all handled correctly with clear messages.
- **Role-based access control** (REQ-02, REQ-07, REQ-08) is correctly implemented — members cannot see or access librarian-only features.
- **Search & filter** (REQ-03) is fully functional and case-insensitive as required.
- **Overdue detection** (REQ-06) correctly flags records past their due date.
- **Record isolation** (REQ-08) is correctly enforced — members can only view their own borrow history.

### System Weaknesses

- **Borrow limit enforcement** (REQ-04) is completely absent in the backend — the most critical business rule is not implemented.
- **Email validation** (REQ-07) is insufficient — the server-side validation does not enforce proper domain format.
- **Error message clarity** (REQ-07) needs improvement — duplicate email should return a specific, distinct error message.
- **Overdue notification** (REQ-05) on return is missing — reduces librarian awareness of late returns.

---

## 7. Priority Recommendations — Bonus B4

The following fixes are recommended in priority order based on defect severity, business impact, and effort to fix.

### Priority 1 — Fix Immediately (Critical / High)

**BUG-04: Enforce 3-Book Borrow Limit**
- **Why urgent**: This is a core business rule. Its absence allows unlimited borrowing, making the library policy unenforceable. Any member can discover this loophole and abuse it.
- **Recommended fix**: Add a server-side check before creating a new borrow record. Query the count of active (non-returned) records for the requesting member. If count ≥ 3, return HTTP 400 with error message "Maximum borrow limit of 3 books reached."
- **Estimated impact**: High — resolves a critical functional gap.

**BUG-02: Enforce Proper Email Format on Server-Side**
- **Why urgent**: Invalid emails are stored in the database, breaking downstream processes (notifications, account recovery) and compromising data integrity.
- **Recommended fix**: Apply a proper regex validation on the backend (e.g., `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`). The client-side Flutter validation should also be aligned with this rule.
- **Estimated impact**: High — prevents corrupt data from entering the system.

### Priority 2 — Fix in Next Sprint (Medium)

**BUG-01: Show Overdue Warning on Book Return**
- **Why important**: Librarians need immediate awareness when overdue books are returned to enforce late-return policies and update membership records.
- **Recommended fix**: In the return-book handler, check if the borrow record's status is "Overdue" before completing the return. If overdue, include an additional line in the success notification: "Note: This book was overdue by [N] days."
- **Estimated impact**: Medium — improves librarian workflow without blocking core functionality.

### Priority 3 — Fix When Convenient (Low)

**BUG-03: Clarify Duplicate Email Error Message**
- **Why useful**: This is a UX improvement. The current message misleads librarians into thinking they typed the email incorrectly, when in fact the email is already registered.
- **Recommended fix**: Catch the unique-constraint violation from the database separately and return the message: "This email address is already registered in the system."
- **Estimated impact**: Low — improves usability without functional impact.

---

## 8. Overall Conclusion

The Library Management System demonstrates solid implementation of authentication, role-based access, search, and record isolation features. However, the **failure to enforce the 3-book borrow limit** (BUG-04) represents a critical gap that directly contradicts a stated business requirement in the SRS. Combined with the email validation issue (BUG-02), the system currently has data integrity and business rule compliance risks that should be addressed before any production release.

**Recommendation**: The system should **not be released** until BUG-04 and BUG-02 are resolved. BUG-01 and BUG-03 can be addressed in a follow-up patch.

| Overall Quality Assessment | ⚠️ Needs Improvement |
|---|---|
| Functional Correctness | 85.2% (23/27 TCs pass) |
| Business Rule Compliance | ❌ Fails (critical limit not enforced) |
| Data Integrity | ❌ At risk (invalid email accepted) |
| UX / Error Messages | ⚠️ Needs improvement |
| Release Readiness | ❌ Not recommended without fixes |
