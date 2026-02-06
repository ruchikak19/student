---
layout: post
title: Ruchika Kench Individual Blog Skill B
categories: ['N@tm']
comments: False
breadcrumb: True
permalink: /indiv_skillB_blog/
date: 2026-02-06
---

## Badges Backend — AP CSP Performance Task (Skill B) Ruchika Kench
## Project Overview
This project has a badge achievement system using a transactional database.
Users earn badges as they progress, and the system records, retrieves, and analyzes badge data using a many-to-many relationship.

The program uses:
- Flask (backend API)
- SQL to make/display the transactional tables
- Transactional table (user_badges and badges)
- JSON-based input/output
- Lists generated dynamically from database queries
- No hard-coded badge or user data (all dynamic!)

---

## Prompt 1: Program Purpose and Function

### Program Purpose
The purpose of this program is to:
- Store badge definitions in a database
- Track which badges each user has earned
- Award badges securely to authenticated/logged-in users
- Display badge progress and leaderboard rankings

---

## How the Program Functions
- Badge data is stored in the badges table.
- When a user earns a badge, a UID is added to the user_badges transactional table in the Users with badge column.
- Lists generated from database queries are used to calculate progress and rankings.

<img width="1856" height="566" alt="Image" src="https://github.com/user-attachments/assets/24f9addf-f104-429e-8daa-0f56685dc38f" />

<img width="1849" height="513" alt="Image" src="https://github.com/user-attachments/assets/c58a8f82-d1e6-4110-a3d1-8e8451f27973" />

---

## Input
Input is provided through a POST request with JSON data.

### Example Input (POST body)
```json
{
  "badge_id": "perfect_prompt_engineer"
}
```

Additional input:
The authenticated user data

---

## Output
The program outputs structured JSON data.

### Example Output
```json
{
  "success": true,
  "message": "Badge \"Perfect Prompt Engineer\" awarded!",
  "new_badge": true
}
```

---

## Prompt 2a: Data Abstraction (List Usage)

### List Initialization (Database Query)

```python
user_badges_objs = UserBadge.query.filter_by(
    user_id=current_user.id
).all()
```

This statement returns a list of UserBadge objects.
Each element represents one badge earned by the user.

### How the List Manages Complexity
The program does not need to track badges individually.
The list grows or shrinks automatically based on database content.
The same logic works for all users and all badge counts.

---

## Prompt 2b: Managing Complexity

### Without Using a List
Each badge would require:
- Separate variables  
- Separate conditional checks  
- Leaderboards would require manually counting each badge.

### With Using a List
- A single loop processes all badges.
- Badge logic does not change as new badges are added.
- Complexity is reduced by grouping related data.

---

## Prompt 2c: Procedure and Algorithm

### Procedure Definition 

#### Award Badge Procedure
```python
@badge_api.route('/award', methods=['POST'])
@token_required()
def award_badge():
```

### What this procedure does
- Accepts badge input from the user  
- Checks if the badge exists  
- Prevents duplicate awards  
- Creates a transaction record  
- Returns a JSON response  

### Procedure Parameters
```python
body = request.get_json()
badge_id = body.get('badge_id')
current_user = g.current_user
```

- badge_id determines which badge to award  
- current_user identifies which user receives the badge  
These parameters allow the procedure to be reused for all users and badges.

---

## Algorithm Explanation (Sequence, Selection, Iteration)

### Step 1: Sequence — Retrieve Badge
```python
badge = Badge.query.filter_by(
    _badge_id=badge_id
).first()
```
- Retrieves badge metadata from the database.
- If the badge does not exist, the program stops.

---

### Step 2: Selection — Validate Input
```python
if not badge:
    return jsonify({
        'error': 'Invalid badge_id'
    }), 400
```
- Boolean condition checks whether the badge exists.
- This prevents invalid database operations.

---

### Step 3: Selection — Prevent Duplicate Badges
```python
existing = UserBadge.query.filter_by(
    user_id=current_user.id,
    badge_id=badge.id
).first()

if existing:
    return jsonify({
        'success': True,
        'message': 'Badge already earned',
        'new_badge': False
    }), 200
```

- Ensures each badge can only be earned once which ensures database accuracy.

---

### Step 4: Sequence — Create Transaction Record
```python
user_badge = UserBadge(
    user_id=current_user.id,
    badge_id=badge.id
)
user_badge.create()
```

- Inserts a new row into the user_badges table.
- This represents a successful badge award (shown by a popup in the FE)

---

## Iteration Example — Badge Progress

### Iteration Over Badge List
```python
all_badges = Badge.query.all()

for badge in all_badges:
    progress.append({
        'id': badge._badge_id,
        'name': badge.name,
        'earned': badge.id in earned_badge_ids
    })
```

- The loop checks each badge dynamically (validity of badge awarded)
- Works regardless of how many badges exist.

---

## Prompt 2d: Testing and Debugging

### Error Identified
Users could receive the same badge multiple times.

### Cause
No check existed for previously earned badges.

### Fix Implemented
```python
if existing:
    return jsonify({'new_badge': False})
```

### Testing Process
I used Postman to:
- Send repeated POST requests with the same badge ID  
- Verify only one database record is created  
Also checked through the FE (does the same badge show up in the sidebar more than once or not??)

---

## PPR Questions Quick code snippet summary

### Procedure & Selection

#### First Conditional Statement
```python
if not badge_id:
    return jsonify({'error': 'badge_id is required'})
```

- Boolean expression checks for missing input.
- If false, the procedure exits safely.

---

## Procedural Abstraction

### How Parameters Manage Complexity
- The same procedure handles all badge types.
- No separate code is needed for each badge because of the transactional data format

---

## Procedure Calls & Testing

### Procedure Call Example 1
```json
{ "badge_id": "delightful_data_scientist" }
```

### Procedure Call Example 2
```json
{ "badge_id": "responsible_ai_master" }
```

Different inputs trigger different execution paths because badges are awarded according to different requirements in different submodules

---

## Logic Error Example

### Modification That Causes a Logic Error
```
Removing the duplicate check
if existing:
return ...
```

### Effect on Output if I removed this section :(
- Duplicate badges appear.  
- Leaderboard counts become inaccurate.

---

## List Utilization (Required)

### List Initialization
```python
user_badges_objs = UserBadge.query.filter_by(
    user_id=current_user.id
).all()
```

### List Use
```python
user_badges = [
    ub.badge.read()
    for ub in user_badges_objs
]
```

Converts database records into output-ready data.

---

## Final Summary
This program satisfies AP CSP Skill B by:
- Using multiple backend procedures  
- Accepting POST input and returning JSON output  
- Using lists generated from database queries  
- Applying selection and iteration  
- Avoiding hard-coded data  