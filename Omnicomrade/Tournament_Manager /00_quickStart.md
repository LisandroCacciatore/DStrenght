# QUICK_START.md

# Trainer Assistant — Quick Start Guide

Estimated setup time: 2–3 hours

---

## Phase 0 — Preparation

1. Create a new Google Sheet named "TrainerAssistant_Template"
2. Open Extensions → Apps Script
3. Rename project to "Trainer Assistant"

---

## Phase 1 — Create Database Sheets

Create the following sheets:

* DB_Programs
* DB_Routines
* DB_Sessions
* DB_Exercises
* DB_Athletes
* DB_Coaches
* CONFIG

Add initial CONFIG values:
CURRENT_WEEK_START | <today>
DEFAULT_SESSION_DURATION | 90
PLATE_LOADING_KG | 25,20,15,10,5,2.5,1.25

---

## Phase 2 — Basic Menu

Inside Code.gs add a custom menu:
Trainer Assistant →

* New Program
* New Routine
* Schedule Session
* Today Panel
* Test Installation

---

## Phase 3 — Minimum Viable Flow

1. Create Program
2. Create Routine
3. Schedule Session
4. Open Day Panel
5. Mark Attendance

If these steps work, you have a functional MVP.

---

## Next Step

Implement performance logging and weekly export.
