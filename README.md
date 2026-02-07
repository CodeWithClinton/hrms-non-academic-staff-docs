
## 1) List My Appraisals (Paginated)

**GET** `my_appraisals/`

Returns a paginated list of appraisals belonging to the logged-in staff, ordered by latest updates.

### Query Params (optional)

| Param             | Type           | Example          | Meaning                       |
| ----------------- | -------------- | ---------------- | ----------------------------- |
| `status`          | string         | `draft`          | Filter by appraisal status    |
| `approval_status` | string         | `pending`        | Filter by approval status     |
| `completed`       | boolean string | `true` / `false` | Filter by completion          |
| `period_id`       | int            | `3`              | Filter by appraisal period id |

> Note: `completed` is only applied if it is exactly `"true"` or `"false"`.

### Response (200)


Example (typical pagination shape):

```json
{
  "count": 12,
  "next": "https://api.example.com/my_appraisals/?page=2",
  "previous": null,
  "results": [
    {
      "id": 10,
      "staff": 4,
      "staff_name": "John Doe",
      "appraisal_period": 2,
      "period_name": "2025 Annual Review",
      "status": "draft",
      "approval_status": "pending",
      "self_comments": "...",
      "completed": false,
      "score": "0.00",
      "rating": null,
      "date_created": "2026-02-01T10:00:00Z",
      "updated_at": "2026-02-02T09:00:00Z",
      "submitted_at": null,
      "documents": []
    }
  ]
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`

---

## 2) My Appraisal Stats

**GET** `my_appraisal_stats/`

Returns summary counts for the logged-in staff’s appraisals.

### Response (200)

```json
{
  "my_appraisal_stats": {
    "total_appraisals": 12,
    "draft": 3,
    "submitted": 5,
    "reviewed": 4,
    "completed": 7,
    "approval_status": {
      "pending": 6,
      "approved": 4,
      "rejected": 2
    }
  }
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`

---

## 3) Apply for Appraisal (Create or Update for Active Period)

**POST** `apply_for_appraisal/`

Creates an appraisal for the **currently active appraisal period** (based on today’s date), or updates the existing one for that period.

### How active period is chosen

Backend finds the latest `AppraisalPeriod` where:

* `is_active = true`
* `start_date <= today <= end_date`

If none exists, request fails.

### Request Body

Supports:

* `self_comments` (optional)
* `documents` (optional) → **multiple files**

#### Option A: JSON (no files)

`Content-Type: application/json`

```json
{
  "self_comments": "My self appraisal notes..."
}
```

#### Option B: Multipart (with files)

`Content-Type: multipart/form-data`

Form fields:

* `self_comments`: text (optional)
* `documents`: file (optional, repeatable)

Example:

* `documents`: file1.pdf
* `documents`: file2.jpg

### Behavior

* If appraisal for active period **does not exist** → creates it with:

  * `status = "draft"`
  * `approval_status = "pending"`
* If it **already exists**:

  * If `self_comments` is provided (even empty string), it updates it.
  * It can also attach uploaded `documents`.

### Response

* **201 Created** if created
* **200 OK** if updated

```json
{
  "message": "Appraisal created successfully",
  "appraisal": {
    "id": 10,
    "staff": 4,
    "staff_name": "John Doe",
    "appraisal_period": 2,
    "period_name": "2025 Annual Review",
    "status": "draft",
    "approval_status": "pending",
    "self_comments": "My self appraisal notes...",
    "completed": false,
    "score": "0.00",
    "rating": null,
    "documents": [
      {
        "id": 1,
        "file": "https://api.example.com/media/appraisals/doc1.pdf",
        "uploaded_at": "2026-02-02T09:00:00Z",
        "created_at": "2026-02-02T09:00:00Z",
        "updated_at": "2026-02-02T09:00:00Z"
      }
    ]
  }
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`
* **400** `{ "error": "No active appraisal period found" }`

---

## 4) Get My Appraisal Detail

**GET** `get_my_appraisal_detail/<appraisal_id>/`

Returns one appraisal record **only if it belongs to the logged-in staff**.

### Path Param

* `appraisal_id` (int)

### Response (200)


```json
{
  "id": 10,
  "staff": 4,
  "staff_name": "John Doe",
  "appraisal_period": 2,
  "period_name": "2025 Annual Review",
  "status": "draft",
  "approval_status": "pending",
  "self_comments": "...",
  "completed": false,
  "score": "0.00",
  "rating": null,
  "documents": []
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`
* **404** If appraisal doesn’t exist OR doesn’t belong to this staff

---



## 1) List My Publications (Paginated)

**GET** `my_publications/`

Returns the logged-in staff’s publications ordered by newest `publication_date`.

### Query Params (optional)

| Param       | Type           | Example          | Meaning                                                           |
| ----------- | -------------- | ---------------- | ----------------------------------------------------------------- |
| `approved`  | boolean string | `true` / `false` | Filter by `approval_status`                                       |
| `type`      | string         | `journal`        | Case-insensitive filter by publication type                       |
| `q`         | string         | `ai`             | Searches `title`, `publisher`, `lead_author`, `research_keywords` |
| `date_from` | date string    | `2025-01-01`     | Filter where `publication_date >= date_from`                      |
| `date_to`   | date string    | `2025-12-31`     | Filter where `publication_date <= date_to`                        |

Notes:

* `approved` only applies if exactly `"true"` or `"false"`.
* `date_from/date_to` must be `YYYY-MM-DD`. If parsing fails, that filter is ignored.

### Response (200)

Example (typical pagination shape):

```json
{
  "count": 4,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 9,
      "staff": 4,
      "staff_name": "John Doe",
      "title": "Deep Learning for X",
      "type": "Journal",
      "approval_status": true,
      "approved_by": 2,
      "approved_by_name": "Prof. Jane Smith",
      "publisher": "ACM",
      "issn_isbn": "1234-5678",
      "volume_issue_page": "Vol 2, Issue 1, pp 10-20",
      "lead_author": "John Doe",
      "co_authors": "A. One, B. Two",
      "role_in_publication": "Lead Author",
      "affiliated_department": "Computer Science",
      "research_keywords": "AI, Deep Learning",
      "abstract": "....",
      "publication_file": "https://api.example.com/media/publications/paper.pdf",
      "proof_of_acceptance": null,
      "dataset": null,
      "publication_date": "2025-08-14",
      "doi": "10.1234/abcd",
      "institutional_repository_ids": null,
      "research_area_clusters": null,
      "crossref_included": false
    }
  ]
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`

---

## 2) Create Publication

**POST** `create_publication/`

Creates a new publication for the logged-in staff.

### Important behavior (ownership enforced)

Backend **forces** `staff = request.user.staff.id` regardless of what the client sends.

### Request Body

You can send either:

* **JSON** (no files)
* **multipart/form-data** (recommended if uploading files)

#### Option A: JSON (no files)

`Content-Type: application/json`

```json
{
  "title": "My Research Paper",
  "type": "Journal",
  "publisher": "Elsevier",
  "lead_author": "John Doe",
  "role_in_publication": "Lead Author",
  "affiliated_department": "Computer Science",
  "research_keywords": "AI, NLP",
  "publication_date": "2025-08-14",
  "co_authors": "A. One, B. Two",
  "abstract": "....",
  "issn_isbn": "1234-5678",
  "doi": "10.1234/abcd",
  "institutional_repository_ids": {"openalex": "W123"},
  "research_area_clusters": ["AI", "NLP"],
  "crossref_included": false
}
```

#### Option B: Multipart (with files)

`Content-Type: multipart/form-data`

Form fields (examples):

* `title`, `type`, `publisher`, `lead_author`, `role_in_publication`, `affiliated_department`, `research_keywords`, `publication_date` (required-ish depending on model validation)
* Optional file fields:

  * `publication_file` (file)
  * `proof_of_acceptance` (file)
  * `dataset` (file)

### Response (201)


```json
{
  "id": 10,
  "staff": 4,
  "staff_name": "John Doe",
  "approval_status": false,
  "approved_by": null,
  "approved_by_name": null,
  "title": "My Research Paper",
  "type": "Journal",
  "publisher": "Elsevier",
  "publication_date": "2025-08-14",
  "publication_file": null,
  "proof_of_acceptance": null,
  "dataset": null,
  "crossref_included": false,
  "...": "other fields"
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`
* **400** Validation errors from serializer, e.g. missing required fields:

```json
{
  "title": ["This field is required."],
  "publication_date": ["This field is required."]
}
```

---

## 3) Publication Detail (Retrieve / Update / Delete)

**Route:** `publications/<publication_id>/`

Supported methods:

* **GET** → retrieve one publication
* **PUT** → full update (send all fields you want to keep)
* **PATCH** → partial update (send only fields to change)
* **DELETE** → delete

### 3a) Retrieve Publication

**GET** `publications/<publication_id>/`

#### Response (200)

```json
{
  "id": 9,
  "staff": 4,
  "staff_name": "John Doe",
  "title": "...",
  "approval_status": true,
  "approved_by": 2,
  "approved_by_name": "Prof. Jane Smith",
  "...": "other fields"
}
```

#### Errors

* **404** if not found

### 3b) Update Publication

**PUT/PATCH** `publications/<publication_id>/`

#### Body

* PUT: typically provide all fields you want persisted
* PATCH: provide only changed fields

Example PATCH:

```json
{
  "title": "Updated Title",
  "research_keywords": "AI, ML"
}
```

#### Response (200)

Returns the updated object.

#### Errors

* **400** serializer validation errors
* **404** if not found



### 3c) Delete Publication

**DELETE** `publications/<publication_id>/`

#### Response (200)

```json
{ "message": "Publication deleted successfully" }
```

#### Errors

* **404** if not found

---

## 4) My Publication Stats

**GET** `my_publication_stats/`

Returns summary counts for the logged-in staff’s publications.

### Query Params (optional)

| Param       | Type        | Example      | Meaning                                          |
| ----------- | ----------- | ------------ | ------------------------------------------------ |
| `date_from` | date string | `2025-01-01` | Count only where `publication_date >= date_from` |
| `date_to`   | date string | `2025-12-31` | Count only where `publication_date <= date_to`   |

### Response (200)

```json
{
  "my_publication_stats": {
    "total_publications": 10,
    "approved_publications": 4,
    "pending_publications": 6
  }
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`

---



## 1) Apply for Leave

**POST** `apply_for_leave/`

Submits a leave application for the logged-in staff. Optionally supports uploading multiple documents.

### Required fields

* `leave_type`
* `start_date` (YYYY-MM-DD)
* `end_date` (YYYY-MM-DD)
* `phone_during_leave`

### Optional fields

* `reason`
* `documents` (files, can be multiple — multipart only)

### Request Body

#### Option A: JSON (no documents)

`Content-Type: application/json`

```json
{
  "leave_type": "ANNUAL",
  "start_date": "2026-02-10",
  "end_date": "2026-02-14",
  "phone_during_leave": "08012345678",
  "reason": "Family event"
}
```

#### Option B: Multipart (with documents)

`Content-Type: multipart/form-data`

Form fields:

* `leave_type`: text
* `start_date`: text (YYYY-MM-DD)
* `end_date`: text (YYYY-MM-DD)
* `phone_during_leave`: text
* `reason`: text (optional)
* `documents`: file (optional, repeatable)

Example:

* `documents`: medical_note.pdf
* `documents`: travel_ticket.jpg

### Backend validations

* Dates must parse correctly (`YYYY-MM-DD`)
* `end_date` cannot be earlier than `start_date`
* Prevents overlapping leaves for same staff where existing leave is `PENDING` or `APPROVED` and date ranges overlap

### Response (201)

```json
{
  "message": "Leave application submitted successfully",
  "leave_id": 21,
  "status": "PENDING",
  "current_stage": "HOD",
  "number_of_days": 5
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`
* **400** Missing required fields:

```json
{ "error": "leave_type, start_date, end_date and phone_during_leave are required" }
```

* **400** Invalid date format:

```json
{ "error": "start_date and end_date must be valid dates (YYYY-MM-DD)" }
```

* **400** End date before start date:

```json
{ "error": "end_date cannot be earlier than start_date" }
```

* **400** Overlapping pending/approved leave:

```json
{ "error": "You already have a pending/approved leave overlapping this date range" }
```

---

## 2) My Leave History (Paginated)

**GET** `my_leave_history/`

Returns a paginated list of the logged-in staff’s leave applications ordered by newest first (`-created_at`).


Example (typical pagination shape):

```json
{
  "count": 3,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 21,
      "staff": 4,
      "staff_name": "John Doe",
      "leave_type": "ANNUAL",
      "leave_type_display": "Annual Leave",
      "start_date": "2026-02-10",
      "end_date": "2026-02-14",
      "number_of_days": 5,
      "phone_during_leave": "08012345678",
      "reason": "Family event",
      "status": "PENDING",
      "current_stage": "HOD",
      "is_active": false,
      "documents": [
        {
          "id": 7,
          "leave": 21,
          "file": "https://api.example.com/media/leave_documents/medical_note.pdf",
          "uploaded_by": 4,
          "uploaded_by_name": "John Doe",
          "uploaded_at": "2026-02-07T09:00:00Z"
        }
      ],
      "created_at": "2026-02-07T08:55:00Z",
      "updated_at": "2026-02-07T08:55:00Z"
    }
  ]
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`

---

## 3) My Leave Stats

**GET** `my_leave_stats/`

Returns summary counts for the logged-in staff’s leave applications.

### What it counts

* `total_applied`: all leaves for staff
* `active_leaves`: `APPROVED` leaves currently ongoing (`start_date <= today <= end_date`)
* `pending_leaves`: `PENDING`
* `approved_leaves`: `APPROVED` (total, past + active)
* `rejected_leaves`: `REJECTED`

### Response (200)

```json
{
  "my_leave_stats": {
    "total_applied": 10,
    "active_leaves": 1,
    "pending_leaves": 2,
    "approved_leaves": 7,
    "rejected_leaves": 1
  }
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`




---

## 1) Apply for Promotion

**POST** `apply_for_promotion/`

Creates a promotion request for the logged-in staff and optionally uploads supporting documents.

### Required fields

* `current_rank` (rank id)
* `proposed_rank` (rank id)

### Optional fields

* `session` (id)
* `semester` (id)
* `department` (id)
* `unit` (id)
* `reason` (text)
* `academic_contribution` (text)
* Documents upload:

  * `documents` (files, multiple allowed)
  * `document_names` (optional list of names, same order as `documents`)

### Request Body

#### Option A: JSON (no documents)

`Content-Type: application/json`

```json
{
  "current_rank": 2,
  "proposed_rank": 3,
  "session": 1,
  "semester": 2,
  "department": 5,
  "unit": 9,
  "reason": "Met requirements for promotion.",
  "academic_contribution": "Published papers, supervised students..."
}
```

#### Option B: Multipart (with documents)

`Content-Type: multipart/form-data`

Form fields:

* `current_rank`: 2
* `proposed_rank`: 3
* optional: `session`, `semester`, `department`, `unit`, `reason`, `academic_contribution`
* `documents`: file (repeatable)
* `document_names`: text (repeatable, optional)

Example:

* `documents`: cv.pdf
* `document_names`: Updated CV
* `documents`: publications.zip
* `document_names`: Publications Evidence

Notes:

* If `document_names` is not provided (or shorter than documents), the API uses the file’s original filename for that document’s `name`.

### Response (201)

Returns the newly created promotion request serialized by `PromotionSerializer`.

```json
{
  "message": "Promotion request submitted successfully",
  "promotion": {
    "id": 15,
    "staff": {
      "id": 4,
      "full_name": "John Doe"
    },
    "current_rank": 2,
    "proposed_rank": 3,
    "session": 1,
    "semester": 2,
    "department": 5,
    "unit": 9,
    "reason": "Met requirements for promotion.",
    "academic_contribution": "Published papers...",
    "current_stage": "HOD",
    "status": "SUBMITTED",
    "submitted_at": "2026-02-07T10:00:00Z",
    "reviewed_by": null,
    "reviewed_at": null,
    "review_comment": null,
    "effective_date": null,
    "created_at": "2026-02-07T10:00:00Z",
    "updated_at": "2026-02-07T10:00:00Z",
    "documents": [
      {
        "id": 6,
        "promotion": 15,
        "name": "Updated CV",
        "file": "https://api.example.com/media/promotion_documents/cv.pdf",
        "uploaded_at": "2026-02-07T10:00:01Z",
        "uploaded_by": 4
      }
    ]
  }
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`
* **400** Active request exists:

```json
{ "error": "You already have a promotion request under review" }
```

* **400** Missing required fields:

```json
{ "error": "current_rank and proposed_rank are required" }
```

---

## 2) My Promotion History (Paginated)

**GET** `my_promotion_history/`

Returns paginated promotion requests for the logged-in staff, ordered by newest first (`-created_at`).


### Response (200)


Example (typical pagination shape):

```json
{
  "count": 2,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 15,
      "staff": { "id": 4, "full_name": "John Doe" },
      "current_rank": 2,
      "proposed_rank": 3,
      "status": "SUBMITTED",
      "current_stage": "HOD",
      "submitted_at": "2026-02-07T10:00:00Z",
      "documents": []
    }
  ]
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`

---

## 3) My Promotion Stats

**GET** `my_promotion_stats/`

Returns summary counts for the logged-in staff’s promotion requests.

### What it counts

* `total_applied`: all promotions by staff
* `pending`: promotions with `status` = `SUBMITTED` or `UNDER_REVIEW`
* `approved`: promotions with `status` = `APPROVED`
* `rejected`: promotions with `status` = `REJECTED`

### Response (200)

```json
{
  "my_promotion_stats": {
    "total_applied": 6,
    "pending": 2,
    "approved": 3,
    "rejected": 1
  }
}
```

### Possible Errors

* **403** `{ "error": "Staff profile not found" }`

---


