# Resources and references: DSA Fundamentals

Every source below was opened or its status is stated. Accessed 19 Nov 2026. Verification: **Cross-verified** (two or more independent sources), **Single source**, **Snippet only** (seen in a search result, page not read), **Unverified**.

## Research session log
| Date | Tools available / blocked | Notes |
|---|---|---|
| 19 Nov 2026 | WebSearch and WebFetch worked. Blocked or unreadable: MIT Press product page (403), Uber engineering blog post (404), Google Careers "interview tips" and "how we hire" pages (fetch returned navigation text only, no body), Google 2018 PDF (readable only after downloading it and extracting text locally). | No textbook was read directly; textbook scope comes from publisher/author pages and course syllabi. |

## Sources
| Title | Link | Type | Used for | Status |
|---|---|---|---|---|
| Microsoft Careers: Technical interviewing | https://careers.microsoft.com/v2/global/en/hiring-tips/technical-interviewing | Official company page (undated) | Named topics (arrays and strings, queues and lists, linked lists, trees and tries, hash maps and sets, graphs, recursion, n log n sorting, Big O); 45-minute round; no pseudocode; test your own code | Opened. Single primary source for Microsoft |
| Google: Preparing for Google Technical Internship Interviews (2018 PDF) | https://services.google.com/fh/files/misc/preparing_for_google_technical_internship_interviews.pdf | Official company document, **2018, internship audience** | Named topics (hash tables, trees and tries, heaps, graphs, recursion, sorting, Big-O, OS concurrency, discrete math and counting, NP-complete recognition); think aloud; clarify; brute force then improve; real code in C, C++, Go, Java, JavaScript or Python | Opened (text extracted locally). Dated and for interns; treat as indicative |
| Uber: How we hire | https://jobs.uber.com/en/what-moves-us/how-we-hire/ | Official company page (© 2026, undated) | Stages: apply, recruiter, hiring manager, technical interview (tech roles), role-dependent exercise, team interview, decision; technical round is a collaborative problem in real time | Opened. No topic list on the page |
| Google Careers: Interviewing at Google (interview tips) | https://www.google.com/about/careers/applications/interview-tips/ | Official company page | Intended for current guidance | **Could not read** (navigation text only) |
| Uber Engineering: engineering interview process | https://www.uber.com/blog/engineering-interview-process/ | Company engineering blog | Coding interview details | **Not read** (404). A search snippet mentioned CodeSignal or a whiteboard and "real-world" problems; treat as Unverified |
| Apple interview process | (none found) | n/a | n/a | **No official Apple source found.** Only Apple Community forum posts (anecdotal) and job postings that list "algorithm and data structure analysis" for software roles (snippet only). Do not assume an Apple format |
| MIT OpenCourseWare 6.006 Introduction to Algorithms, Spring 2020: Syllabus | https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/pages/syllabus/ | University course (primary) | Scope (dynamic arrays, heaps, balanced BSTs, hash tables, sorting, graph searching, dynamic programming); prerequisites (Python programming, discrete math); CLRS as optional reference | Opened. Lecture topic order seen in search results only |
| MIT 6.006 lecture notes | https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-spring-2020/pages/lecture-notes/ | University course | Primary study material for the teaching sequence | Listed in search results; not yet read in detail |
| Python documentation: Time complexity of operations on built-in types | https://docs.python.org/3/builtins/time-complexity.html | Official docs (primary) | Costs of list, dict, set operations; caveats (amortised, hash assumptions, CPython only) | Opened. Cross-verified with cppreference for hash-table average and worst cases |
| cppreference: Containers library | https://en.cppreference.com/w/cpp/container | Reference documentation | Container categories; O(log n) for ordered associative, O(1) average and O(n) worst for unordered | Opened (last ~7 KB unread) |
| cppreference: std::sort | https://en.cppreference.com/w/cpp/algorithm/sort | Reference documentation | O(N log N) comparisons, not stable | Opened. Single source for C++ sort; verify against the ISO standard before relying in exams |
| cppreference: std::unordered_map | https://en.cppreference.com/w/cpp/container/unordered_map | Reference documentation | Average O(1); buckets; worst case from the standard (not stated on the page) | Opened |
| Algorithms, 4th Edition (Sedgewick and Wayne): book site | https://algs4.cs.princeton.edu/home/ | Textbook site | Chapter scope (fundamentals, sorting, searching, graphs, strings); lectures and exercises exist; code is in Java | Opened |
| Introduction to Algorithms, 4th ed. (Cormen, Leiserson, Rivest, Stein; MIT Press, 2022) | https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/ | Textbook | Reference text for proofs and depth | Publication details from library and retailer listings in search results; publisher page returned 403; chapter list **not verified** |
| Stanford CS161 Design and Analysis of Algorithms | https://web.stanford.edu/class/archive/cs/cs161/cs161.1236/index.html | University course | Cross-check of scope (analysis, sorting, data structures, divide and conquer, DP, greedy, graphs) | Snippet only; not opened |

## Cross-verification results
- **Core interview topics (hash tables, trees, graphs, recursion, n log n sorting, Big-O, linked lists, arrays and strings):** named by both Google (2018 PDF) and Microsoft (page). Cross-verified for these two companies only.
- **Hash table costs:** Python docs and cppreference agree: O(1) average, O(n) worst case when keys collide. Cross-verified.
- **Heaps / priority queues:** Google PDF names them; the Microsoft page does not. Single source.
- **Tries:** both name them (Microsoft "trees and tries", Google "trie-trees"). Cross-verified.
- **Dynamic programming, greedy, backtracking, union-find, bit manipulation, intervals:** **not named by either company page or the Google PDF.** MIT 6.006 covers DP. Their appearance in interviews rests on unofficial reports, which this research did not verify. We will teach DP because it is standard algorithm design and is widely reported, but I will not claim any company's official list includes it.
- **NP-completeness recognition and discrete math / counting:** Google PDF only.
- **Operating systems concurrency:** Google PDF only; separate subject, not in this one.

## Conflicts, version differences, outdated material
- Company documents are undated or old (Google's is from 2018 and aimed at interns). Formats have likely changed; verify the current loop with a recruiter.
- `std::sort`'s worst case was O(N log N) only from C++11 onward in practice (cppreference notes a defect-report fix applied retroactively); older sources may claim O(N^2) worst case. Python's `list.sort` is documented as O(n log n) worst case; its stability is not stated on the page opened and must be confirmed from the Python Sorting HOWTO before being taught as fact.
- Python's `list.index` complexity is not listed on the time-complexity page; do not teach a figure without verifying.

## Could not verify / could not access
- Apple's interview process (no official source).
- Current Google interview-prep content on careers pages.
- Uber's coding-interview blog details.
- Full table of contents of CLRS 4th edition.
- Company-specific question lists: none will be provided; any such list would be unofficial.
