CLAUDE.md — Whilst Project
This file gives Claude Code full context on the Whilst project so it can work autonomously without needing to re-explain the codebase.


________________


What is Whilst?
Whilst is a Critical Design product — not a finance app, but a mirror. It uses spending tracking as an entry point to ask harder questions about time, money, and what people actually value in their lives.


Core insight: Most people spend the most money on the things they care about least. Most people spend the most time longing for the things they won't spend money on.


Tagline: This is not a finance app. It is a mirror.


Design philosophy: Trojan Horse design — finance as the entry point, soul-searching as the real product. Every purchase is converted not into hours worked, but into distance from what the user said matters most.


________________


Repository Structure
whilst/                     (GitHub: uxcircles/whilst, private)


├── index.html              Landing page (light theme, bilingual)


├── app/index.html          Main app (dark theme, bilingual) — served at whilst.work/app


├── og.png                  OG image for social sharing


└── favicon.png             App favicon


Deployed at: https://whilst.work (GitHub Pages, custom domain via Namecheap)
App URL: https://whilst.work/app


________________


Tech Stack
* Frontend: Single-file HTML (no framework, no build step)
* Fonts: Newsreader (serif) + DM Mono (monospace) from Google Fonts
* Backend: Supabase (project: aeyrdjyowwopkluckjlb, region: eu-west-2)
* Auth: Supabase email auth
* Hosting: GitHub Pages
Supabase Config
URL: https://aeyrdjyowwopkluckjlb.supabase.co


Anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFleXJkanlvd3dvcGtsdWNramxiIiwicm9sZSI6ImFub24iLCJpYXQiOjE3Nzc1NzA2NjYsImV4cCI6MjA5MzE0NjY2Nn0.SIJQ2ozMfvJKeDR8JwZWMd4ome5QPrUhmtN6gZOnLG4
Supabase Tables
* profiles — extends auth.users, auto-created on signup
* onboarding — user's answers to all onboarding questions
* expenses — spending records with motive, date, isEscaping flag
* weekly_answers — monthly reflection text entries
* monthly_answers — monthly soul question answers
* followup_queue — 7-day follow-up queue for escapist purchases


All tables have Row Level Security enabled. Users can only access their own data.


________________


Design System (whilst.html)
Colours
--void: #0a0a0a        /* background */


--white: #f5f0e8       /* primary text */


--soft: #c8b89a        /* secondary text */


--dim: #7f7f7f         /* tertiary text */


--muted: #3a3a3a       /* disabled/placeholder */


--border: #1e1e1e      /* borders */


--accent: #c8a96e      /* gold — primary action colour */


--danger: #8b3a3a      /* delete/warning */
Typography
* Newsreader — editorial serif, used for all questions, narratives, large text
* DM Mono — monospace, used for labels, stats, system UI elements
Button Conventions
* .btn-primary — outline gold button (splash screen Begin)
* .submit-expense — full-width outline gold button (Record, Continue, etc.)
* .report-submit — text-only gold button (Note it quietly →)
* .begin-again-btn — small text gold button (secondary actions)
* .fab — floating action button (+ Record, fixed bottom-right)
* All buttons: gold by default, hover → white
Key CSS Notes
* body::after — grain texture overlay, z-index: 9
* .onboard-nav — fixed footer for onboarding back/continue, z-index: 10000
* .screen.active — no overflow-y (would break fixed positioning)
* visualViewport listener handles keyboard pushing content up


________________


i18n System (whilst.html)
const userLocale = navigator.language || 'en-GB';


const isZh = userLocale.startsWith('zh'); // all zh* → Traditional Chinese


const t = { key: isZh ? '中文' : 'English', ... };


function applyTranslations() { /* updates all DOM elements */ }


* All UI strings are in the t object
* applyTranslations() is called on page load
* Dynamic JS content (renderWeekly, renderAnnual, ob2 fields) also checks isZh inline
* Monthly questions have two separate arrays: monthlyQuestions (EN) and monthlyQuestionsZh (ZH)
* Dates use Intl.DateTimeFormat(userLocale) for locale-aware formatting


________________


App Architecture (whilst.html)
Screens
screen-splash → screen-ob1 → screen-ob2 → screen-ob3 → screen-app
Onboarding Flow
ob1: What is the one thing you most dread missing?


* Options: person / freedom / work / other


ob2: Dynamic questions based on ob1 answer:


* person → who (parent/child/partner/other) → calculates remaining days/hours
* freedom → last acted / escape fund amount / what's stopping you
* work → what is it / hours per week / when last done
* other → limiting factor / money or time dimension / respective fields


ob3: Life % slider + monthly salary
State Object
let state = {


  longing: '',              // what they most dread missing (text)


  windowDays: 0,            // calculated remaining days/hours with person


  windowPerson: '',         // who the window refers to


  escapeFund: 0,            // freedom: target escape fund


  workHoursPerWeek: 0,      // work: hours per week


  otherDimension: '',       // other: 'money' or 'time'


  otherFund: 0,             // other/money: target fund


  lifePercent: 40,          // what % of life is truly yours


  hourlyRate: 8.33,         // calculated from monthly salary


  selectedRelation: '',     // parent/child/partner/other


  partnerLivesTogether: undefined,


  salarySet: false,


  expenses: [],


  weeklyAnswers: [],


  monthlyAnswers: [],


  followupQueue: [],


};
Key Functions
* goToApp() — async, saves onboarding, calls populateApp()
* populateApp() — renders all app data, updates account section
* commitExpense(name, amount, motive) — async, saves to localStorage + Supabase
* renderExpenses() — renders ledger
* renderWeekly() — renders monthly narrative (bilingual)
* renderAnnual() — renders annual report (bilingual)
* updateStats() — updates 4 stat cards, shows distress note if needed
* deleteExpense(id) — async, removes from localStorage + Supabase
* initAuth() — non-blocking, runs after splash
* loadUserData() — loads all Supabase data, redirects to app
* saveOnboarding() — syncs onboarding to Supabase
* applyTranslations() — applies all i18n strings to DOM
Conversion Logic by Longing Type
* person → % of remaining days with that person
* freedom → % of escape fund
* work → hours that could have been spent on loved work
* other/money → % of target fund
* other/time → % of remaining days


________________


App Tabs (whilst.html)
Now Tab
* North star card (shows state.longing + window calculation)
* 4 stat cards: spent this month / life hours / escaping % / forgotten %
* Distress note (appears when escaping ≥ 60% or forgotten ≥ 70%)
* Expense ledger with delete button
* FAB button (+ Record) — fixed bottom-right, only shows on Now tab
Mirror Tab
* This Month — narrative report + weekly reflection textarea
* The Question — monthly soul question (rotates from 24-question pool)
* This Year — annual report + life % slider + begin again section


________________


Modals (whilst.html)
* Record modal — full-screen slide-up, z-index: 150
* Followup modal — 7-day check-in for escapist purchases, z-index: 200
* Salary modal — triggered after first expense if salary not set, z-index: 200
* Auth modal — sign in / create account with password toggle, z-index: 300
* Begin again modal — soul question gate before resetting, uses .followup-modal


________________


Landing Page (index.html)
Light theme (--paper: #f7f3ec), same Newsreader + DM Mono fonts.
Sections
1. Hero — two-column grid, title + app mockup
2. Insight — core philosophy quote
3. Features — 4 numbered features
4. Dark section — first onboarding question as CTA
5. Substack — essay link card
6. Footer
i18n
Same isZh detection. All strings in strings object, applyTranslations() on load.
Nav
Frosted glass: background: rgba(247,243,236,0.75) + backdrop-filter: blur(16px)


________________


Important Design Decisions
* No PWA (skipped intentionally for now)
* No em dashes in Chinese copy (use commas instead)
* Full uppercase only for system labels (THE LEDGER, NOW, MIRROR)
* Normal case for questions and action buttons (Why, honestly?)
* Gold (#c8a96e) for all interactive elements, hover → white
* whilst.html links to whilst.work/whilst.html (not root, landing page is root)
* OG image: https://raw.githubusercontent.com/uxcircles/whilst/main/og.png


________________


Common Tasks
Adding a new translation key
1. Add to const t = { ... } object: key: isZh ? '中文' : 'English'
2. Use t.key in JS or add document.getElementById('id').textContent = t.key in applyTranslations()
Adding a new onboarding field
1. Add field to state object
2. Add HTML in the relevant ob2 config template literal
3. Add label translation to t object and placeholder to ph_* keys
4. Add to saveOnboarding() upsert object
5. Add to loadUserData() restoration block
6. Add to confirmBeginAgain() reset block
Deploying
git add .


git commit -m "your message"


git push origin main


# GitHub Pages auto-deploys, whilst.work updates in ~1 minute
Checking for issues
Run node --check whilst.html — won't work directly (HTML not JS), but you can extract the script block and check it.


________________


Pending Items
* Customise Supabase email templates (Auth → Email Templates in Dashboard)
* Add app screenshots/mockup to landing page hero
* Consider PWA manifest for add-to-homescreen
* Test auth flow end-to-end on live domain
* Publish Substack articles (ZH + EN versions ready in repo)
* Update Substack URL in index.html once published