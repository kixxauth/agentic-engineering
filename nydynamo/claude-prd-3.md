# NY Dynamo Hockey Website

## Information Architecture & URL Structure

**Project Goal:** Redesign the NY Dynamo website to attract new players to try out for the organization's teams.

**Key Audiences:** Prospective players, parents/families considering the program

**Primary Content Pillars:** Team information, coaching philosophy, tryout details, success stories, testimonials, photos/videos, and news updates

---

## Site Structure Overview

### Homepage (`/`)
Main landing page designed to attract new players with:
- Hero section introducing NY Dynamo and its mission
- Tryout information and call-to-action
- Program overview highlighting key differentiators
- Featured testimonials from current players/parents
- Recent news/updates
- Links to explore programs by category
- Call-to-action buttons for interested prospective players

---

## Primary Navigation Menu

1. **Home** → `/`
2. **Programs** → `/programs` (landing page for all program types)
3. **Tryouts** → `/tryouts` (consolidated tryout information)
4. **Teams** → `/teams` (team directory and filtering)
5. **About** → `/about` (organization info, coaching philosophy, values)
6. **Contact** → `/contact` (contact form, location, hours)

---

## URL Structure by Section

### Programs Section (`/programs`)
Landing page with overview of all program categories and ability to filter/explore

#### Youth Programs (`/programs/youth`)
- Overview of 9U through 18U age groups
- Difference between Elite (AAA) and Platinum (AA) tiers
- Program philosophy and development focus

**Sub-pages by age group:**
- `/programs/youth/9u` - 9U teams
- `/programs/youth/10u` - 10U teams
- `/programs/youth/11u` - 11U teams
- `/programs/youth/12u` - 12U teams
- `/programs/youth/13u` - 13U teams (note: also shows as 13O in current system)
- `/programs/youth/14u` - 14U teams
- `/programs/youth/16u` - 16U teams
- `/programs/youth/18u` - 18U teams

Each age group page shows:
- Program description
- Both Elite and Platinum team options
- Coaches for each team with profiles
- Links to individual team pages

#### Girls Programs (`/programs/girls`)
- Overview of girls-specific offerings
- Dedicated girls program philosophy
- Age groups and tiers

**Sub-pages by age group:**
- `/programs/girls/10u` - 10U girls
- `/programs/girls/12u` - 12U girls
- `/programs/girls/14u` - 14U girls
- `/programs/girls/16u` - 16U girls

Each girls program page shows:
- Program overview
- Team listings
- Coach information
- Success stories specific to girls program

#### Juniors/Premier Programs (`/programs/juniors`)
- NCDC and USPHL Premier Juniors information
- Player development pathway to college
- Coaching staff bios
- Team schedules and results

**Sub-pages:**
- `/programs/juniors/ncdc` - NCDC Juniors division
- `/programs/juniors/usphl-premier` - USPHL Premier division

---

### Teams Section (`/teams`)
Master team directory with filtering and search capabilities

#### Team Detail Pages
Dynamic URL structure for individual teams:
- `/teams/[season]/[age-group]/[team-name]`
- Example: `/teams/2025-2026/14u-elite/14u-elite-team-a`

**Each team page includes:**
- Team roster with player profiles (names, numbers, positions)
- Coach profiles with bios and coaching philosophy
- Schedule (games and practice times)
- Game results and statistics
- Photo gallery from games/practices
- Team news and updates
- Parent/family resources for that team

---

### Tryouts Section (`/tryouts`)
Central hub for all tryout information

**Sub-pages:**
- `/tryouts` - Master tryout page with overview
  - Current season tryout dates and times
  - How to register for tryouts
  - What to expect at tryouts
  - Requirements and eligibility
  - FAQ about the tryout process

- `/tryouts/register` - Tryout registration form/application
  - Interest form for prospective players
  - Age group selection
  - Skill level/experience
  - Contact information
  - Submit to prospective player database

- `/tryouts/youth` - Youth program tryout details
  - Specific dates for each age group
  - Location and logistics
  - Age/birth year requirements
  - What to bring
  - Program tiers explained (Elite vs Platinum)

- `/tryouts/girls` - Girls program tryout details
  - Girls-specific program information
  - Tryout dates for each age group
  - Program structure and opportunities

- `/tryouts/juniors` - Juniors/Premier tryout details
  - NCDC and USPHL Premier opportunities
  - Eligibility and requirements
  - Tryout timeline and process

---

### About Section (`/about`)
Organization information and philosophy

**Sub-pages:**
- `/about` - Main about page
  - Organization mission and values
  - Brief history of NY Dynamo
  - Tier 1 status and affiliations
  - Links to affiliated organizations (THF, AHF, AGHF, NCDC, USPHL)

- `/about/coaching-philosophy` - Program approach
  - Training methodology
  - Player development pathway
  - What makes NY Dynamo different
  - Coaching staff development

- `/about/coaching-staff` - Coaching directory
  - Head coaches by program/team
  - Assistant coaches
  - Support staff
  - Staff bios and experience
  - Photos

- `/about/success-stories` - Alumni and achievements
  - College commitments
  - Professional/advanced opportunities
  - Player testimonials
  - "Where are they now" alumni profiles

---

### News & Content Section (`/news`)
Blog and updates section (consider for Phase 2)

**Sub-pages:**
- `/news` - News listing page
  - Recent announcements
  - Blog posts about programs
  - Team updates
  - Event announcements

- `/news/[post-title]` - Individual news articles

---

### Contact Section (`/contact`)
Contact and location information

**Sub-pages:**
- `/contact` - Main contact page
  - Contact form
  - Location and venue information
  - Hours of operation
  - Email addresses by department
  - Phone numbers
  - Social media links
  - Map to Capital Arena

---

### Additional Pages

#### Store (`/store`)
- Link to/embed NY Dynamo equipment store
- Current: `/posts/2025-summer-store` (consider reorganizing)

#### Billeting Program (`/billeting`)
- Information about host families
- Billet family questionnaire link
- Player billeting questionnaire link
- FAQ about billeting

#### Resources (Optional, Phase 2)
- `/resources/parent-guides` - Guides for parents
- `/resources/player-development` - Training resources
- `/resources/faqs` - Frequently asked questions

---

## Key Content Elements by Page Type

### Program/Team Pages (High Priority)
- Coach profile photos and bios
- Team action photos and videos
- Schedule and game results
- Player testimonials
- Parent testimonials
- Program philosophy/approach

### Tryout Pages (High Priority)
- Clear, prominent registration CTAs
- Dates, times, and locations
- Requirements and eligibility
- What to expect/prepare for
- FAQ section
- Contact information for questions

### About/Coaching Philosophy (High Priority for Recruiting)
- Why choose NY Dynamo?
- Coaching methodology
- Success metrics and alumni outcomes
- Staff qualifications
- Affiliations and certifications

---

## Content Organization Principles

1. **Multiple Pathways:** New visitors can explore by age group, gender, program level, or through tryout funnel
2. **Clear CTAs:** Tryout registration and contact forms available throughout
3. **Visual Content:** Heavy use of photos/videos on program and team pages to showcase quality and energy
4. **Testimonials:** Student and parent quotes strategically placed on recruiting pages
5. **Social Proof:** Alumni success stories, affiliations, and certifications prominently displayed
6. **Mobile-Friendly:** All pages optimized for mobile browsing (parents accessing on phones)

---

## Phase Implementation Recommendation

**Phase 1 (MVP):**
- Homepage redesign
- Programs section (youth, girls, juniors overviews)
- Team directory and individual team pages
- Tryout information and registration
- About/Coaching philosophy
- Contact page

**Phase 2 (Enhancement):**
- News/blog section
- Coaching staff directory
- Success stories/alumni section
- Billeting program pages
- Resource center

**Phase 3 (Advanced Features - Low Priority):**
- Team statistics dashboards
- Live scoring integration
- Parent portal/login area
- Video gallery
- Advanced search/filtering