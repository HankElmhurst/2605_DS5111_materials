# Online MSDS Syllabus Summer 2026

## DS5111: Software and Automation Skills
**Syllabus: Summer 2026**

---

## Course Information  
**Course Start Date–End Date:** May 18–August 6, 2026  
**Credit Hours:** 3 
**Prerequisites:**
* Basic understanding of python programming and building machine learning models.
* A github account (you'll need to pass on your github handle in one of the first quizzes)
* A windsurf account.  The pricing may have changed, so I will circle back on this requirement.
**Catalog Description:**
```
Code an end-to-end data science project with core software engineering and automation to
quickly integrate into a corporate environment. Use version control to focus on solutions,
leverage automation at your command line and in the cloud, deliver solid code by
incorporating testing, lower extension and maintenance time with OOP and Design Patterns,
ensuring your code's path to production to deliver a complete package to the enterprise.
```

**Live Online Meeting Times:** Tuesdays, 8:30 – 9:30 pm EST  

---

## Instructor Information
|                  | Instructor                     | Instructional Support                 |
| --               | --                             | --                                    |
| Name             | Efrain Olivares                | Karolina Naranjo-Velasco              |
| Email            | dpy8wq@virginia.edu            | kn3se@virginia.edu                    |
| Phone            | 805 452 5838                   |                                       |
| Office Hours     | [by Zoom XXX day/time]         |                                       |
| Office Location  | Remote                         | Remote                                |
| Teams/Slack      | Channel TBD                    |                                       |


---

## Course Overview  
```
Course DS5111: Streamlining Data Science with Software and Automation Skills

This is a very "hands-on" course.  In essence it's a learning lab.  It's purposeis to
compliment your Data Science skills by walking through putting a pipeline together in as
close a way as you would in a real work setting.

Our emphasis will be on removing as much of the 'surprise' factor when entering a real work
setting, as well as picking up skills that make you efficient.

For efficiency's sake we'll take a close look at using the command line as it is still, after
all these years, the backbone of how software is configured, accessed and run remotely.  To
that end, we'll work on Ubuntu AWS instances throughout the semester to get comfortable with
using the cloud.

Tool-wise, we'll focus heavily on on tools like Github and Github-Actions.  Both of these
are both a major pain point and a real boost for productivity, respectively.   You are just
about guaranteed to use one or both of these settings in any technical job these days.
We'll also look at a Data Build Tool, (DBT), a very common python package for data
manipulation.  Finally we'll also spend some time with Snowflake, a data-lake/store tool
that is ubiquitous and usually paired with DBT .

In terms of practices we'll touch on Object Oriented Design,  Testing and Design Patterns.
The primary focus on these is for efficiency.  It is the ideas and experience you gain from
learning how to incorporate these practices into your workflow that pay off by saving you
time and energy.  This applies regardless of the programming language or environment you
eventually end up working on.

Our project will consist of gathering stock market data for our 'startup employer'.  By
the end of the course you should have assembled a pipeline in the automatically collects
data from Yahoo/Wall-street-journal, processes it with DBT, stores it in Snowflake, and
then presents insight data in a report.
```

---

## Learning Outcomes  
By the end of the semester, you will be able to:

1. Work in github, (back out a commit, work with branches)
1. Set up and work in a cloud based linux box on AWS
1. Use makefiles for quick automation at the command line
1. Use cron to set up repeating jobs
1. Supplement your python programming with asserts and tests
1. Set up Github-Action to auto run  your tests in the cloud
1. Understand what Object Oriented Programming is and when to use it
1. Understand what Design Patterns for software are all about
1. Set up Data Build Tool to process the raw data  with from your cron jobs
1. Push that data into Snowflake for storage and further processing in SQL
1. Generate report tables

---

## Semester Schedule 

| Week | Tuesday | Topic                                | Due  | 
| --   | --      | --                                   | --   |
|  1   | 5/19    | Github/AWS Account Setup             | 6/1  |
|  2   | 5/26    | VMs, git, makefiles, collecting data | 6/8  |
|  3   | 6/2     | Testing                              | 6/15 |
|  4   | 6/9     | Github Actions                       | 6/22 |
|  5   | 6/16    | Github rebase/merge; cron jobs       | 6/29 |
|  6   | 6/23    | Object Oriented Programming          | 7/6  |
|  7   | 6/30    | Design Patterns                      | 7/13 |
|  8   | 7/7     | Data Build Tool (DBT)                | 7/20 |
|  9   | 7/14    | Set up Jupyter lab/windsurf          | 7/27 |
|  10  | 7/21    | Snowflake                            | 8/3  |
|  11  | 7/28    | Docker                               | 8/7  |
|  12  | 8/4     | Parametrized Testing                 | 8/7  |

---

## What to Expect in This Fully Online Course  

This course is delivered in 12 learning modules through Canvas. Each course module consists of a live session, a lab, a quiz and a reading assignment.  There will also be two journals, one midway through the course, and one at the end.

Meetings (video and teleconference) for live lessons will be held on Zoom.  A good part of the live session time will include live coding.

Students are expected to bring up their coding environment on AWS and come to class prepared to participate in each week’s live coding activity.

Aside from the live course, office hours for both the instructor and the TA will be made availble to help with any questions or technical issues you may run into.  Note that there are videos provided to go over a lot of the steps for setup and for the homework.  However, as things move fast in this space, I will be adding videos as we move through the class to supplement some of the new material.

After live meetings, students should focus on the reading, and the assigned lab for the week.  Students are encouraged to work in groups as long as every student turns in his own work and states who they worked with.  Towards the second half of the course we'll also form teams of 4 or 5 people and have a SCRUM meeting every week, this will be for the purpose of giving you a glimpse of the Agile meetings that are standard in the software industry.  More details on that later, but they should only take 10 to 15 minutes to complete during the week.

The journal is intended to capture your reflections on the material.  The primary intent is to spark some exploration into the material.  You should treat it more like a "lab side experiments" section.  Good entries would include ways you explored the linux environment beyond the lab, or perhaps adding more to the automation we do with github actions, or even just repeating difficult section.  Anything that shows you've taken the material and pushed the boundaries a little bit further is fair game.  You don't need to do anything fancy, as long as you turn in the journal you'll get the points.

Teams channel.  Students will be expected to participate in the Teams channel as a way to ask quetions asynchronosly and help other students.  Both the instruction and the TA will also participate, and there may be questions only they can answer, but by and large the forum is for students to work together and help each other.  For this course, your participation is required as it better approximates real working conditions in industry.

---

## Required Software  
You will be provided access to AWS resources and all of the software we need is installable or available for free online.

**Python**  
We'll install and run python on our AWS EC2 instances.  

**Command-line Interface**  
You'll be provided access to an AWS instance where you will use the linux command line (ubuntu).
For direct ssh access you'll also need any moderatly powered laptop.
Mac or Linux come with access to a linux command line.  **Important**:  For Windows users, please install
the linux ubuntu subsystem.  [Installing the WSL on Windows](https://learn.microsoft.com/en-us/windows/wsl/install) 

Other option we will explore are connecting to your AWS instance via Jupyter Lab and Windsurf.

**Docker**  
You will need a dockerhub account.  You can create a free one here [hub.docker.com](https://hub.docker.com/)  

**Software**  
* Windows users who want to ssh directly can install the Windows linux subsystem as mentioned above.

Additionally there are terminal apps that help make the experience smoother.
* Iterm2 for Mac
* Gitbash for Windows

We will also use windsurf to connect remotely, you can download it at [windsurf.com](https://windsurf.com/)

---

## Learning Resources  

### Required Textbook/Resources  
Textbook readings will typically come from one source, a condensed text online, with references to other external sources.
You will see links to these reading in the module section.  The source books are many and the class github repository will have the full list.

Links:  
- https://myuva-my.sharepoint.com/...  
- https://webaim.org/techniques/hypertext/link_text#text  

---

* DS5111 Software And Automation Skills for Data Science
Instructor Name: Efrain Olivares
Last Updated: May 18, 2026

---

## See an Error in the Course Materials?  

I strive to build out this course with the right links and complete and error-free information so that you can learn as much as possible outside of our class time. If you see anything – a broken link, missing data, etc., please let me or the course instructional assistant know, or use the "Report Course Errors" form in Canvas.

---

## Grading  

### Grade Breakdown  

| Assignment         | Percentage of Overall Grade | Points     |
|----------          |---------------------------  |--------    |
| Lab Submissions    | 45%                         |   200      |
| Quizzes            | 45%                         |   200      |
| Journals           |  5%                         |    20      |
| Class Partcipation |  5%                         |    20      |
| **Total**          | **100%**                    | **440**    |

---

## Grading Policies

### Expectations

The expectation is that all lab assignments, live coding, quizzes, and journals will be submitted on time, by the due date and time/time zone, indicated in Canvas.  There are some independent labs, but for the most part they build on each other so it is important to keep up or else you may find yourself rushing through a ton of work in the end.  A lot of the skills we'll cover require time to practice them, they are no difficult per se, but the best outcome is that they become muscle memory and that can only come from spaced repetition.

---

### Late Work Policy

We recognize you are juggling jobs, lives, and course work. Given the fast pace of this online MSDS program, we also recognize there may be times when you need an extension.  Each lab and quiz is due one week from the live lecture, however, you'll note the actual due date is set for two weeks.  Please use this sparingly second week sparingly.

#### Two Automatic Extensions Policy

In the event that the provided time alloted for an assignment is still not enough, (any number of life situations, illness, moving, deployment etc. can come up) we can provide Two automatic extensions to help you manage your priorities and not fall behind with your learnin. This policy allows you to submit your homework or other assignments up to 7 days beyond the two week normal span, three times, during the term without penalty or question. In order to enact your extension, you need to notify the instructor or the TA (preferably both) BEFORE the two week standard period elapses so we can plan accordingly.

---

## Extenuating Circumstances

If you need help, are falling behind and struggling to catch up, or if extenuating circumstances arise, the sooner you reach out for help, the more options there are to help and support you. Please contact Kristy Simpkins, kms8k@virginia.edu, Assistant Director, Online Student Success and Engagement, if you are experiencing extenuating circumstances.

---

## Accessing Grades and Feedback in Canvas

In this course we'll rely heavily on github, so course feedback will typically be through pull requests.  We will strive to provide feedback at about a week after an assignment was due.  If you need feedback sooner than that, you can come to office hours and we can go over your questions.

---

## Weekly Attendance at Live Sessions

Your partiipation in live class is required as we will in most classes do some live coding together.  Attendance, and participation in the Teams channel will count for the participation grade.  If you can not make it to a live session please email us to let us know, then watch the recorded video and follow along with the exercise we did in class.

https://myuva-my.sharepoint.com/:w:/g/personal/hxa7zd_virginia_edu/ERtRWUVsQYVHoxPCW87-WZwBnQjmKmtG_eh8igP-BLXEGA?e=zMB6DN  

---

## Grade Scale

According to the SDS Grading Policies, the standing of a graduate student in each course is indicated by one of the following grades: A+, A, A-; B+, B, B-; C+, C, C-; D+, D, D-; F.  

B- is the lowest satisfactory grade for graduate credit. You will be graded according to the following markings:

| Grade | Range                           |
|------ |------                           |
| A+    | > 97                            |
| A     | 93 - 97                         |
| A-    | 90 - 92.9                       |
| B+    | 87- 89.9                        |
| B     | 83 – 86.9                       |
| B-    | 80 – 82.9 Minimum passing grade |
| C+    | 77 – 79.9                       |
| C     | 73 – 76.9                       |
| C-    | 70 – 72.9                       |
| D+    | 67 – 69.9                       |
| D     | 63 – 66.9                       |
| D-    | 60 – 62.9                       |
| F     | 0 – 59.9                        |

---

## Course Policies

### Communication

---

### Zoom Expectations

Don’t let Zoom keep you from being an active learner!  

You are required to have your camera on for the duration of our live class sessions. Your virtual presence adds tremendous value to the course and program: it allows me as your instructor to gauge understanding; it helps you focus at a time when we know you are tired after working a long day; it creates connections with you and your peers.  

You may create a background using the Zoom Virtual Background Generator, though it is not required. Distractions will happen – a needy child, wandering cat, messy pile – and I understand this as a working professional myself.  

Mute yourself unless you want to speak. Use the “Raise Hand” feature if you wish to speak. Your questions and comments are valuable, and I encourage you to participate often.  

Set your name to what you would like to be called. Update your profile with your preferred name. Be sure to log in with your UVA account.

---

## Recording of Live Zoom Class Sessions

Class sessions for this course will be recorded. Recordings will be available only to the instructor(s) and students enrolled in the class, including those who may miss a live session. Recordings will be deleted when no longer necessary. Recordings may not be reproduced, shared with those not enrolled in the class, or uploaded to other online environments. Students who are not comfortable with participating in a recorded discussion session should contact the instructor to request an alternate assessment activity. Students in the class are prohibited from recording of any kind unless authorization is obtained from the instructor.


https://web3.darden.virginia.edu/backgroundgenerator/index.html  

---

## Communication and Student Response Time

Email or slack me whenever you want, and I will respond as soon as possible. Email is usually the preferred mode of communication although you can use Teams as well. Asking questions in front of others promotes discussion and reduces me having to repeat myself. 
However, please be mindful that not everyone is at the same pace.  For those of you that tend to jump ahead please do not ask questions about a future module in class, let's use that time to keep everyone who is following along on target.  You can come to office hours and we can talk about future module questions if there is time.   You can also start a discussion in Teams and keep that as a thread for a specific forward question.  

Questions containing personal information should be emailed to me. Throughout our time together, the sooner you inform me of any problem that may affect your attendance or performance, the better the chance we have of solving it together.

---

## Teams

[LINK TO TEAMS WILL BE UPDATED HERE]

---

## Academic Integrity: Collaboration and Cheating


This course is a little different than the others you've taken.  The goal of the course is to pass on skills that will help you land on your feet in a new job and keep you focused on the Data Science.  The aim is to reduce the 'surprise effect' by introducing you to a lot of tools and 'gotcha's'.  I know some of the topics we'll cover are wide and one lecture isn't barely enough to scratch the surface.  What this translates to is that in order to pass the course you need to participate, even if that means things do go as planned.  Here are some things I'd like to point out from previous experience teaching this course.

* If you find yourself spending more than an hour on any one aspect of a lab, STOP.  Reach out to other students and a question in Teams, or Email me or the TA.
* For this class, it is more valuable to spend time using the tools and going through the motions.  What I'm aiming for is that you start to feel comfortable with some of these tools so that when you encounter them in a new job you'll be more confident having had seen them before.  And more importantly, you'll stay focused on data science and not unpacking a new tool from scratch.
* The use of tools like chatgpt, gemini, claude are encouraged, but with the caveat that they should not be used to replace your work.  In other words, if you do that you won't learn anything.  The tools are better used, (and we'll have examples in class) as a force multiplier once you have some knowledge of a topic.

---

## SDS Guidelines on AI Tools and Assistance

The use of generative AI tools and foundation models, (i.e. ChatGPT, DALL-E, Stable Diffusion, Midjourney, GitHub Copilot, and similar tools) is permitted with the following activities in accordance with the stated guidelines at no penalty:

• Brainstorming and refining your ideas;  
• Fine tuning your research questions;  
• Finding information on your topic;  
• Drafting an outline to organize your thoughts; and  
• Checking grammar and style.  

Students are responsible for  

• Acknowledging that large language models tend to produce incorrect facts and fake citations.  
• Acknowledging that code generation models may produce inaccurate outputs.  
• Acknowledging that image generation models can occasionally come up with highly offensive products.  
• Taking responsibility for any inaccurate, biased, offensive, or otherwise unethical content submitted, regardless of the origin (i.e. student-generated or from a foundation model).  
• Properly citing the contribution of the foundation model or other AI tools in submitted material.  
• The entirety of any information they submit, based on an AI query or AI assistance.  

The use of generative AI tools is NOT permitted for the following activities:

• Impersonating students in classroom contexts, such as by using the tool to compose discussion board prompts or content entered into a Zoom chat.  
• Completing group work assigned to a student, unless it is mutually agreed upon that they may utilize the tool.  
• Writing a draft of a writing assignment.  
• Writing entire sentences, paragraphs or papers to complete class assignments.  

Students may be penalized for  

• Using a foundation model without including an acknowledgement.  
• Improperly citing the use of work by other human beings or the submission of work by other human beings as that of the student.  
• Violating intellectual property laws.  
• Submitting materials containing misinformation or unethical content.  

The usage of AI tools must be properly cited to stay within university policies on academic honesty. Failure to adhere to these guidelines will result in a failing grade on the assignment or exam (a zero) and may be an honor code violation depending on the context (to be determined at the instructor’s discretion). Having said all these disclaimers, the use of foundation models is encouraged, as it may make it possible for you to submit assignments with higher quality, in less time.

---

## Technical Support

We have standardized on the use of AWS EC2 instances to reduce the amount of time we spent debugging may different OSs.  About the only caveat is the installation of the linux subsystem on Windows.  Outside of that we can't provide technical support if you are running in a different environment.

### Technical Support Contacts

• Please rech out to the instructor or the TA for technical support if you can not log into or something is wrong with your EC2 instance.

---

## School of Data Science Support and Policies

### Office of Student Support

The Office of Student Support is here to support your academic journey toward success. Their office provides resources to help you be successful and engaged. If you need support resources related to student success or personal well-being, please reach out to the Office of Student Support directly at: SDS_OSS@virginia.edu.

Please also utilize the Data Science Student Portal for resources and information about career, academic support, engagement opportunities, funding, and more.

---

### Peer Tutors

The School of Data Science offers free, virtual, one-on-one tutoring with MSDS alumni. Our peer tutors provide personalized support tailored to your needs, whether you're struggling with a specific concept or want to deepen your understanding. They can clarify doubts, reinforce learning, and guide you toward success. Use the SDS Peer Tutors Bookings site to schedule your session.

---

### Online MSDS Student Advisory Board (OSAB)

Online MSDS students have a student advisory board to help them navigate their online experience and engage with peers about opportunities and concerns related to online learning at the School of Data Science. You can contact them at SDS_OSAB@virginia.edu.

---

### Career Services

The School of Data Science Career Services Team provides a wealth of opportunities for you to learn, connect, and grow. These offerings are adapted to the needs of the cohort or the year and may take different forms as needs change.

Complete your profile and career interests, explore the Data Analytics Resource Card or make an appointment with Career Services at the School of Data Science by using your UVA Email to Login to Handshake.

---

### Graduate Record

Visit the School of Data Science Graduate Record for policies and information about academic regulations, academic standing, financial assistance, and grades.

---

## University Policies

### University Email Policy

Students are expected to activate and then check their official UVA email addresses on a frequent and consistent basis to remain informed of University communications, as certain communications may be time sensitive. Students who fail to check their email on a regular basis are responsible for any resulting consequences.

---

### Academic Integrity and University of Virginia Honor System

The School of Data Science relies upon and cherishes its community of trust. We firmly endorse, uphold, and embrace the University's Honor principle that students will not lie, cheat, or steal, nor shall they tolerate those who do. We recognize that even one honor infraction can destroy an exemplary reputation that has taken years to build. Acting in a manner consistent with the principles of honor will benefit every member of the community both while enrolled in the School of Data Science and in the future. Students are expected to be familiar with the university honor code, including the section on academic fraud.

All work should be pledged in the spirit of the Honor System of the University of Virginia. The instructor will indicate which assignments and activities are to be done individually and which permit collaboration. Students who submit exams electronically acknowledge the Honor Pledge by agreeing to the following statement:

• “On my honor, I have neither given nor received aid on this examination, nor did I have prior knowledge of its contents.”

For more information, please visit UVA Honor Committee.

Course Evaluations: Student feedback is critical to the school, the instructor, and future students. Students are expected to complete anonymous and confidential course evaluations in a timely manner for each course at the end of each term.

---

### Discrimination/Harassment/Retaliation

UVA prohibits discrimination and harassment based on age, color, disability, family medical or genetic information, gender identity or expression, marital status, military status (which includes active duty service members, reserve service members, and dependents), national or ethnic origin, political affiliation, pregnancy (including childbirth and related conditions), race, religion, sex, sexual orientation, veteran status. UVA policy also prohibits retaliation. All faculty and TAs are also responsible employees for disclosures or reports of potential discrimination, harassment, and retaliation.

---

### Disability and Pregnancy Accommodations

If you anticipate or experience any barriers to learning in this course, please discuss your concerns with me. If you have a disability, or think you may have a disability, contact the Student Disability Access Center (“SDAC”) to request reasonable accommodation(s) for this course. If you have accommodations through SDAC, send me your Faculty Notification Letter as soon as possible and meet with me so we can develop an implementation plan together.

Students may be entitled to reasonable accommodations for pregnancy, childbirth, or related medical issues. Please contact SDAC for additional information. Pregnant and parenting students are encouraged to contact SDAC or EOCR to discuss plans and ensure ongoing access to their academic courses and program. Information for pregnant and parenting students is also available on EOCR’s Pregnancy and Parenting Resources webpage.

---

### Student Mental Health and Wellbeing

The University of Virginia is committed to advancing the mental health and wellbeing of its students, while acknowledging that health and well-being are tied to academic success and goal attainment.

Between classes, deadlines, careers, and family responsibilities, it can feel like there’s never a good time to take care of yourself. That’s why online MSDS students have access to free, virtual, 24/7 counseling and psychiatric care through TimelyCare! Use your NetBadge to get started at timelycare.com/uva.

---

### Additional Resources

The School of Data Science Office of Student Support can help find resources for students experiencing Emergency Needs. Please contact them at SDS_OSS@virginia.edu.

---

### Additional Resources  
SDS_OSS@virginia.edu  

---
