# Moodle Quick Guide: Dropboxes & H5P Content

A reference for setting up assignment dropboxes and adding H5P content to a Moodle course (Moodle 4.x / 5.x; notes for older versions where they differ).

---

## Contents

1. [Before you start](#before-you-start)
2. [Part 1 — Add a dropbox (Assignment activity)](#part-1--add-a-dropbox-assignment-activity)
3. [Part 2 — Upload H5P content to the Content bank](#part-2--upload-h5p-content-to-the-content-bank)
4. [Part 3 — Add H5P to the course as a graded activity](#part-3--add-h5p-to-the-course-as-a-graded-activity)
5. [Troubleshooting](#troubleshooting)
6. [Recommended settings checklist](#recommended-settings-checklist)

---

## Before you start

- **Role:** You need the **Teacher** (editing teacher) role on the course. If you don't see *Edit mode* or the activity options described below, ask your site admin to update your role.
- **Terminology:** Moodle has no activity literally named "Dropbox." A dropbox is an **Assignment** activity.
- **H5P files:** If you built your H5P on h5p.org, Lumi, or another tool, export/download it as a **`.h5p`** file first.

---

## Part 1 — Add a dropbox (Assignment activity)

### Steps

1. **Turn on Edit mode**
   Open the course and switch on the **Edit mode** toggle (top-right).
   *Older versions:* click **Turn editing on**.

2. **Pick a section**
   Scroll to the week/topic where the dropbox belongs and click **+ Add an activity or resource**.

3. **Choose Assignment**
   Select **Assignment** from the activity picker.

4. **Fill in the basics**
   | Field | What to enter |
   |---|---|
   | Assignment name | e.g., "Week 3 Lab Submission" |
   | Description | What students should submit and how |
   | Additional files | Optional template, rubric, or starter file |

5. **Set availability dates**
   | Setting | Meaning |
   |---|---|
   | Allow submissions from | When the dropbox opens |
   | Due date | Deadline — late submissions still accepted (and flagged late) |
   | Cut-off date | Optional hard stop — no submissions after this |
   | Remind me to grade by | Optional reminder for you |

6. **Configure submission types**
   - ☑ **File submissions** — for uploads
   - ☐ **Online text** — if students type directly into Moodle
   - Set **Maximum number of uploaded files**
   - Set **Maximum submission size**
   - Set **Accepted file types** (e.g., `.pdf, .docx`)

7. **Optional settings**
   - **Feedback types:** feedback comments, feedback files, annotate PDF
   - **Submission settings:**
     - *Require students to click the submit button* (recommended — see checklist)
     - *Additional attempts* (allow resubmission)
   - **Grade:** point value, scale, or *None* for ungraded

8. **Save**
   Click **Save and return to course** or **Save and display**.

### What students see

Students open the activity and click **Add submission**, upload their file(s), then **Save changes** (and **Submit assignment** if you require it).

---

## Part 2 — Upload H5P content to the Content bank

The **Content bank** stores H5P content so it can be reused across activities and kept up to date in one place.

### Steps

1. Open your course.
2. In the course navigation bar (*Course · Settings · Participants · Grades · Reports · More*), go to **More → Content bank**.
   *Moodle 3.9–3.11:* **Content bank** is in the left navigation drawer.
3. Click **Upload** and choose your `.h5p` file.
4. Moodle opens a **preview** — confirm the content plays correctly.
5. *(Optional)* From the content's page:
   - **Rename** it to something clear
   - Set **Make public / Make unlisted** to control whether other teachers can see it
   - **Edit** it directly in Moodle (if the content type is supported)

---

## Part 3 — Add H5P to the course as a graded activity

Embedding the file is not enough to track progress — use the **H5P activity** so Moodle records attempts and scores.

### Steps

1. Turn on **Edit mode** and go to the target section.
2. Click **+ Add an activity or resource → H5P**.
   > Choose **H5P** (the built-in activity), **not** "Interactive Content" (the older third-party plugin).
3. Enter a **Name** (and optional description).
4. Under **Package file**, click **Add** → in the file picker, choose **Content bank** (left side) → select your content.
5. When prompted, choose how to attach it:
   | Option | Behavior | When to use |
   |---|---|---|
   | **Link to the file** (alias) | Updates in the Content bank flow through to the activity | ✅ Usually best |
   | **Make a copy** | Activity has its own frozen copy | When you want a version that won't change |
6. Configure **Grade** and **Attempts options**:
   - Enable attempt tracking
   - Grading method: highest, average, first, or last attempt
   - *(Only relevant if the H5P contains scored questions.)*
7. Click **Save and display** and test it as a student would.

### Viewing results

Open the H5P activity → **Attempts report** to see each student's attempts and scores. Grades also flow into the course **Gradebook**.

---

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| No **Edit mode** toggle | Role is student or non-editing teacher | Ask admin for the **Teacher** role on the course |
| No **Upload** button in Content bank | Missing content bank permissions | Ask admin to grant `moodle/contentbank:upload` |
| H5P preview shows an error | Content type not installed/enabled or outdated | Admin: *Site administration → H5P → Manage H5P content types* — enable or update |
| H5P plays but no scores recorded | Content was embedded in a page/text editor instead of added as an activity | Re-add it using **Add an activity → H5P** |
| Students' drafts count as submitted | "Require submit button" is off | Edit assignment → *Submission settings* → turn it on |
| Can't find "Content bank" under **More** | Older Moodle or theme difference | Check the left navigation drawer, or ask your admin |

---

## Recommended settings checklist

**Assignment dropbox**
- [ ] Turn on **Require students to click the submit button** so drafts don't count as submitted and students aren't confused at the deadline
- [ ] Restrict **Accepted file types** to what you actually want to grade
- [ ] Set a **Cut-off date** if late work shouldn't be accepted
- [ ] Enable **Feedback comments** (and *Annotate PDF* if you grade PDFs)

**H5P**
- [ ] Upload to the **Content bank** first
- [ ] Add it as an **H5P activity** (not an embed)
- [ ] Choose **Link to the file** so future edits update automatically
- [ ] Turn on **attempt tracking** if the content has scored questions
- [ ] Test as a student (or with *Switch role to → Student*) before releasing
