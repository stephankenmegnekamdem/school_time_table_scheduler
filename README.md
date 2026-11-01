# school_time_table_scheduler

A console program written in Java that builds a weekly timetable for six classes. It asks for the school's classes, subjects, required hours, teachers, teacher absences, class teacher assignments, and teaching eligibility. It then places the fixed lessons (religion, social hour, and arts), generates the remaining lessons with a randomized greedy search, and validates the result against every rule.

## Table of contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Running the program](#running-the-program)
- [Using the program](#using-the-program)
- [How the timetable works](#how-the-timetable-works)
- [Fixed lessons](#fixed-lessons)
- [Scheduling rules](#scheduling-rules)
- [Output](#output)
- [Project structure](#project-structure)
- [How the program works internally](#how-the-program-works-internally)
- [Troubleshooting](#troubleshooting)
- [Known limitations](#known-limitations)
- [Status](#status)
- [License](#license)

## Features

- Interactive console input for classes, subjects, hours, and teachers
- Class teacher assignment, with one class teacher per class
- Teacher absences by time slot
- Teaching eligibility per teacher, class, and subject
- Automatic placement of fixed lessons: Rel and Socialhour for every class, Arts for classes 2 to 6
- Randomized search with up to 1000 attempts
- Validation of fixed lessons, teacher conflicts, absences, daily subject limits, exact subject hours, and eligibility
- Printing of the full timetable with fixed lessons marked
- Printing of the best attempt and a score breakdown when no perfect timetable is found

## Requirements

- Java 8 or newer (the code uses lambdas and standard collections only)
- A Java Development Kit (JDK), not just a runtime
- No external libraries
- Optional: IntelliJ IDEA, the IDE the project was written in

To check your installation, run:

```bash
java -version
javac -version
```

Both commands should print a version number.

## Installation

Clone the repository:

```bash
git clone git@github.com:stephankenmegnekamdem/school_time_table_scheduler.git
git clone https://github.com/stephankenmegnekamdem/school_time_table_scheduler.git
cd school-timetable-generator
```


## Running the program

### From IntelliJ IDEA

1. Open the project folder (the one that contains `src`).
2. In the Project panel, right-click `src/Main.java`.
3. Choose **Run 'Main.main()'**.
4. Type your input in the Run console.

### From the command line

From the project folder, compile all sources into an `out` folder and run `Main`:

```bash
javac -d out src/*.java
java -cp out Main
```

On Windows, if `src/*.java` does not expand in your shell, list the files explicitly or use IntelliJ.

## Using the program

The program asks for input in this order.

### 1. Subjects for each class

For classes 1 to 6, the program asks:

1. The number of subjects for the class.
2. For each subject, its name, then the number of hours per week.

Subject names are typed as single words. Use `Math`, not `Mathematics Advanced`, because the program reads input with `next()`, which stops at the first space.

Do not enter `Rel`, `Socialhour`, or `Arts` here. These are placed automatically.

### 2. Teachers

The program asks for the number of teachers. For each teacher it asks, in order:

1. **Name.** Single word, unique across all teachers.
2. **Number of absences**, then the absence slots. Enter each absence as a slot number from 1 to 30. Invalid numbers are rejected and asked again.
3. **Class teacher?** Answer `yes` or `no`. If `yes`, enter the class number (1 to 6). A class that already has a class teacher is rejected.
4. **Number of classes the teacher can teach**, then for each class:
   - the class number (1 to 6),
   - the number of subjects the teacher can teach in that class,
   - for each subject, the subject number from the list shown.

A teacher can be eligible for several subjects in several classes.

### 3. Result

The program prints one of two outcomes:

- `TIMETABLE GENERATION SUCCESSFUL.` followed by the timetable, if every rule is satisfied.
- `NO PERFECT TIMETABLE FOUND.` followed by `BEST AVAILABLE TIMETABLE:`, the timetable, and a `BEST TIMETABLE ANALYSIS` section with the penalty counts.

Finally, it lists the fixed lessons and the number of classes and teachers.

### Example session (abbreviated)

```
Enter the number of subjects for each class
Enter number of subjects for class 1: 2
Enter subject name: Math
Enter number of hours for subject: 4
Enter subject name: English
Enter number of hours for subject: 3
...
Enter the number of teachers
3
Enter teacher name: Anna
Enter the number of absences for Anna
0
Is Anna a class teacher? (yes/no): yes
Which class is Anna the class teacher of? (1-6): 1
...
```

## How the timetable works

- The school week is modeled as a grid of **6 classes × 30 slots**, which is 180 slots in total.
- Each class has 30 slots: **5 days × 6 periods**.
- Slots are numbered 0 to 29 within a class. Slot `0` is Day 1, Period 1. Slot `29` is Day 5, Period 6.
- Day number for a slot is `slot / 6 + 1`. Period number is `slot % 6 + 1`.
- Absences are entered as **slot numbers 1 to 30** and stored internally as **0 to 29**. An absence applies to a single slot, not a whole day.
- All six classes share the same time slots, so a teacher can teach only one class in a given slot.

## Fixed lessons

These are placed before scheduling and are never moved.

| Lesson | Class | Relative slot(s) | Day and period |
|---|---|---|---|
| Rel (religion) | All classes 1–6 | 6 | Day 2, Period 1 |
| Socialhour | All classes 1–6 | 17 | Day 3, Period 6 |
| Arts | Class 2 | 0, 1 | Day 1, Periods 1–2 |
| Arts | Class 3 | 14, 15 | Day 3, Periods 3–4 |
| Arts | Class 4 | 12, 13 | Day 3, Periods 1–2 |
| Arts | Class 5 | 26, 27 | Day 5, Periods 3–4 |
| Arts | Class 6 | 24, 25 | Day 5, Periods 1–2 |

- Rel and Socialhour are taught by the **class teacher** of that class.
- Arts is taught by a teacher named **`Chiara`**. The name is matched without regard to capitalization.
- These placements are hardcoded in `FixedLessonManager.java`. Changing them requires editing that file.

Because of the fixed lessons, the number of free slots per class is:

- Class 1: 28 free slots (2 fixed)
- Classes 2 to 6: 26 free slots (4 fixed)

## Scheduling rules

Every generated lesson must satisfy these rules. The scheduler enforces most of them while placing lessons, and the validator checks them again at the end.

1. **Eligibility.** A lesson's teacher must be eligible for that subject in that class.
2. **No teacher conflicts.** A teacher cannot teach two classes in the same slot.
3. **Absences.** A teacher is never scheduled in a slot where they are absent.
4. **Daily subject limit.** A class may have at most **two** lessons of the same subject on the same day.
5. **Exact hours.** Each subject in each class must appear exactly as many times as its required hours.
6. **Fixed lessons stay put.** Rel, Socialhour, and Arts are placed first and never changed.

Rule 5 is the strictest: a timetable with missing or extra hours is not counted as valid.

## Output

A class in the timetable looks like this:

```
========== CLASS 1 ==========

Day 1
Period 1: Math - Anna
Period 2: EMPTY
Period 3: English - Ben
...
Period 1: Rel - Clara [FIXED]
```

- `EMPTY` means no lesson was placed in that slot. This is only acceptable when a class has more free slots than required hours.
- `[FIXED]` marks Rel, Socialhour, and Arts lessons.

The best-attempt analysis shows:

- `Scheduled lessons`: total filled slots, including fixed lessons
- `Teacher conflicts`
- `Absence violations`
- `Daily subject-limit violations`
- `Missing/extra subject hours`
- `Eligibility violations`

Lower values are better. A valid timetable has all of them at zero.

## Project structure

```
school-timetable-generator/
├── src/
│   ├── Main.java                  Entry point: input, generation, output
│   ├── InputManager.java          Collects classes, subjects, teachers, eligibility
│   ├── School.java                Holds classes, teachers, eligibilities, assignments
│   ├── SchoolClass.java           One class: number, subjects, class teacher
│   ├── Subject.java               Subject name and required hours
│   ├── Teacher.java               Teacher name and absence slots
│   ├── TeachingEligibility.java   Teacher may teach subject in class
│   ├── TeachingAssignment.java    Teacher-class-subject-hours record (not currently used)
│   ├── LessonRequest.java         One required hour of a subject for a class
│   ├── Lesson.java                A placed lesson: subject, teacher, class, fixed flag
│   ├── Timetable.java             The 180-slot grid, lookups, copying, printing
│   ├── TimeSlot.java              Day/period pair (not currently used)
│   ├── FixedLessonManager.java    Places and validates Rel, Socialhour, Arts
│   ├── Scheduler.java             Builds, scores, and repeats timetable attempts
│   ├── TimetableScore.java        Penalty-based score for an attempt
│   └── TimetableValidator.java    Checks all hard rules
├── .gitignore
└── README.md
```

## How the program works internally

1. **Input.** `InputManager.createSchool()` builds a `School` containing six `SchoolClass` objects, the teachers, and the eligibility list.
2. **Fixed lessons.** `FixedLessonManager.addFixedLessons()` places Rel, Socialhour, and Arts into the timetable with `fixed = true`.
3. **Attempt loop.** `Scheduler.generateValidSchedule()` runs up to 1000 attempts:
   - clear all non-fixed lessons,
   - build a new schedule,
   - score it,
   - keep the best-scoring attempt so far,
   - stop early if the validator accepts the schedule.
4. **Building a schedule.** `generateSchedule()`:
   - creates one `LessonRequest` per required hour,
   - sorts requests by difficulty (hardest first), shuffling requests of equal difficulty,
   - places each request in turn.
5. **Placing one request.** `scheduleLessonRequest()`:
   - finds eligible teachers for the class and subject,
   - sorts them by how few free slots they have,
   - picks randomly among the most constrained teachers,
   - lists that teacher's free slots in the class,
   - shuffles them and places the lesson in the first one.
6. **Scoring.** `calculateScore()` counts conflicts, absences, daily-limit violations, eligibility violations, and missing or extra hours. The penalty weights are in `TimetableScore`.
7. **Validation.** `TimetableValidator.isValid()` checks fixed lessons, teacher conflicts, absences, daily limits, exact subject hours, and eligibility.
8. **Output.** `Timetable.printTimetable()` prints the grid, and `Timetable.printFixedLessons()` lists the fixed lessons.

## Troubleshooting

**`Exception: Class N does not have a class teacher`**
Each class needs exactly one class teacher. Restart the program and assign one.

**`InputMismatchException`**
You typed text where a number was expected, such as a class number or hours. Enter whole numbers only.

**`Warning: the timetable may be impossible.`**
Some subject has no eligible teacher. Check the eligibility answers for that class and subject.

**`Fixed lesson validation failed: Chiara was not found.`**
The Arts teacher must be entered with the name `Chiara`.

**`Fixed lesson validation failed: position N is empty` or `is not marked as fixed`**
Fixed lessons were not placed. Check that `addFixedLessons()` runs before scheduling.

**`NO PERFECT TIMETABLE FOUND.`**
Check the analysis section:
- Many missing hours: the subject hours may exceed the free slots, or teacher eligibility or absences are too restrictive.
- Only a few missing hours: try running again, since the search is randomized, or reduce absences or hours.

**Compilation errors after editing**
Make sure every `.java` file is in `src/` and that class names match file names.

## Known limitations

- **Greedy search without backtracking.** Once a lesson is placed, it is never moved. Tight schedules may not be solved even with many attempts.
- **Infeasible data cannot be solved.** If a class needs more hours than free slots, or a subject has too few teachers, no attempt will succeed.
- **Fixed-lesson conflicts are not explained.** If the same teacher is assigned to two fixed lessons at the same time, every attempt is rejected without a specific message.
- **Input is not validated.** Non-numeric input crashes the program. Subject names with spaces break input.
- **Names must be unique.** Some checks compare teacher names and others compare object references.
- **Hardcoded values.** The fixed positions, the Arts teacher name `Chiara`, the six classes, and the 30-slot week are fixed in the code.
- **Unused code.** `TeachingAssignment`, `TimeSlot`, and some methods in `TimetableValidator` and `InputManager` are not used by the current logic.
- **No automated tests.** Correctness is currently checked by running the program on sample input.

## Status

Work in progress. Input, fixed lessons, scheduling, scoring, and validation are implemented. Testing on a complete school dataset is the next step.

## License

No license has been chosen yet. Until one is added, all rights are reserved by the author.

## Author

Stephan Kenmegne Kamdem
