I need to create a new website for a youth hockey organization called "NY Dynamo".

I am not a technical person and do not have any experience in software development.

<about_you>You are an experienced full stack web developer and also have experience as a product manager at a tech company.</about_you>

<step-1>
## Step 1
Ask the user if this is a completely new web development project or a migration of an existing site?
</step-1>

<step-2>
## Step 2
If this is a completely new website then you need to ask the user more questions to establish some user stories. For now, keep this very simple. Just get an idea of what the home page should do. We will iterate on it later and add more pages and functionality to the site.

If we are migrating an existing site then we need to ask the user for the home page link and then crawl the site to determine the information architecture.
</step-2>

<note-1>
Do not plan design tasks or ask the user any questions about the design of the site. We need to get the information architecture figured out first, then we will come back to the design later, in a different context.
</note-1>

<your_job>
## Your Job
Get enough information from the user so that you can create an outline of the site URL structure and information architecture. Once you can develop a document describing the site URL structure and information architecture, then you are done with this job.

NOTE: If this is a completely new website then the only information architecture you will need to establish will be for the home page.

Output the URL structure and information architecture document as markdown text.
</your_job>

---


I need to create a new website for a youth hockey organization called "NY Dynamo".

I am not a technical person and do not have any experience in software development.

<about_you>You are an experienced full stack web developer and also have experience as a product manager at a tech company.</about_you>

I need you to help me craft some user stories and set functional requirements.

<technical_background_info>This website will be built and deployed on a custom managed hosting platform, which includes integrated web development tools.

The website will be built and managed by an experienced software developer and developer operations staff.

The managed hosting platform does *not* provide a login or content management system. But, the developers who use it can manage content in several ways:

- Hardcoding content into HTML templates
- Uploading JSON files of structured data to be rendered by HTML templates
- Uploading markdown files of text content to be rendered by HTML templates

The developers do not not have access to a standard theme library and will be writing all HTML, CSS, and JavaScript by hand.
</technical_background_info>

<your_job>
Your job is to determine what information the developers will need to build this website. You will need to prompt the user with questions to collect the information needed for the developers of the website.

Keep in mind that the user will have information that the developers will need to know, but the user does not know how to express that information. It is your job to ask the user questions to get this information out.

Some examples of hidden information the user has, might be:

- The site might already exist and needs to be migrated to this new custom platform
- There might be branding for the site already available
- Or, this could be a complete greenfield development project

Without a technical background, the user cannot express everything needed to the developers. So, you need to ask the right questions to get all the information from the user that the web developers will need to complete this project.
</your_job>



---

The team page should be at `/seasons/:season_id/teams/:team_id`. It should include:

## Team Name
Example: "NY Dynamo 16U Girls"

## Record
Games played, wins, losses, and ties.

## Roster
- player number
- player name
- player position

## Upcoming Game Schedule
Any scheduled games over the next two weeks (not the full schedule).

Each game should have:

- Date. Ex: Sun, Mar 1, 2026
- Time. Ex: 10:30 AM
- Oppenent. Ex: at Rochester Edge U16
- Location. Ex: Kennedy Arena, 500 West Embargo St, Rome, NY

## Subpages

### Full Schedule

The team full schedule page should be at `/seasons/:season_id/teams/:team_id/schedule`. It should include all scheduled games (75 games at most; no paging).

Each game should have:

- Date. Ex: Sun, Mar 1, 2026
- Time. Ex: 10:30 AM
- Oppenent. Ex: at Rochester Edge U16
- Location. Ex: Kennedy Arena, 500 West Embargo St, Rome, NY

### Game Results

The team results page should be at `/seasons/:season_id/teams/:team_id/game-results`. It should include all past games (75 games at most; no paging).

Each game should have:

- Date. Ex: Sun, Mar 1, 2026
- Result. Ex: Loss 2-3
- Oppenent. Ex: at Rochester Edge U16
- Location. Ex: Kennedy Arena, 500 West Embargo St, Rome, NY



---

1️⃣ Business Goals & Success Criteria

What are the top 3 goals for the NY Dynamo website?

1. Centralize info
2. Improve communication
3. Increas registrations

How does NY Dynamo currently operate?

- *Do they already have a website?* Yes, NY Dynamo already has a website, and some of the information architecture we design here will need to align with the current path structure.
- *Are they using tools like TeamSnap, SportsEngine, Google Drive, email lists, etc.?* Yes, they are using TeamSnap, Bond Sports, Google Drive, and Constant Contact for email lists.

What does “success” look like 6–12 months after launch?
Success can be defined as more recruits and better informed families sending fewer inquiry emails caused by confusion. Parents should demonstrate a higher satisfaction score.

Is this primarily:

- A marketing site?
- An operations portal?
- A community hub?

Or all three? *Primarily a marketing site and operations portal*

2️⃣ Users & Personas

Let’s define who we’re building for. List all user types you can think of:

- Prospective parents
- Current parents
- Players (by age group)
- Volunteers
- NY Dynamo organization admins

Questions:

Who are the primary users? (Top 2–3)

- Prospective parents
- Current parents

Who are secondary users?

- Players (by age group)
- Volunteers
- NY Dynamo organization admins

Do players (ages 8–18) realistically use the site directly, or is everything mediated through parents? *Mostly mediated through parents*

Are there admin roles that need restricted access? *No, the initial version of the site will not have admin access. All data will by synced from 3rd party systems through the backend.*

Are there different teams (travel vs recreational, boys vs girls, different age brackets) that need distinct spaces? *Yes*:

- Elite Co-ed
- Elite Girls
- Platnum Co-ed
- Platnum Girls

3️⃣ Core User Journeys

For a prospective parent, what should they be able to do?

- Learn about the organization?
- View teams by age?
- See costs?
- Register?
- See practice schedule?

For a current parent, what must be easy?

- View schedule?
- Access documents?

For a coach, what matters most?

*Coaches and managers typicall do not use the site*

For a board member/admin, what do they need?

*Organization admins use third part tools like TeamSnap and Bond sports and do not use the site directly for these purposes*

4️⃣ Functional Scope

Tell me yes/no/maybe for each:

Marketing / Public-Facing

- About NY Dynamo: yes
- Teams directory: yes
- Coach bios: yes
- Tryout information: yes
- Registration flow: no
- News/blog: yes
- Photo gallery: no
- Sponsor showcase: no
- Contact form: no
- FAQ: yes

Logged-In Area

*There is no logged-in area yet*

Admin Tools

*There are no admin tools yet. All input data will be coming from third party tools through a backend system*

5️⃣ Constraints & Context

Important for PRD later:

- Budget range? *$10k*
- Timeline? *several weeks*
- Internal tech skills? (Is someone technical maintaining this?) *Yes, we have an experienced web application developer who will build and maintain the site.*
- Do they prefer off-the-shelf (SportsEngine, Wix, Squarespace) or custom build? *This will be a custom build.*
- Any compliance concerns?
    + Youth privacy (COPPA) *none*
    + Payment security *none*
    + Photo permissions *none*

6️⃣ Brand & Positioning

Is NY Dynamo elite/competitive or developmental/community-focused?

*elite/competitive*

Is this local-only or regional travel?

*regional travel*

Is attracting sponsors a major objective?

*No, NY Dynamo does not have sponsors*

Do they have a defined brand identity (logo, colors, voice)?

*Yes, there is a brand identity with a logo and colors (orange and black)*

7️⃣ Pain Points Today

What complaints do you currently hear?

- “I can’t find the schedule.”
- “Tryout info is confusing.”
- “I don’t know who to contact.”
- I don't know who the coaches are
- I can't find the registration forms


---

1. Core Audience and User Roles

- **Parents** will be looking for schedules, registration forms, and ordering uniforms.
- **Recruits** will be browsing the site to see if the NY Dynamo organization is a good fit for them. Good messaging and visual polish will be important to keep them interested.
- **Organization Managers** will use the site to recruit host families for visiting hockey players.

2. Essential Functionality

- **Scheduling** We use a third party system called TeamSnap for managing the schedule. That system provides a way to output the schedule data and sync it back to the new website. There should be a single page which displays all upcoming games for all teams.
- **Registration** Will be done by posting links or embedding forms to third party tools like TeamSnap and Bond Sports.
- **Communication** No realtime communication tools are required
- **Marketing** Need to post embedded YouTube videos and Instagram Posts
- **Team Directory** Each team in the NY Dynamo organization should have a listing in the teams directory. Then, each team should have a dedicated page which lists the team roster and staff. The team page should also include the upcoming games for the next two weeks. And, each team page should link to additional sub pages for the team:
    + The full upcoming schedule
    + Past game results
- **Billeting** When families host hockey players in their home, it is called billeting. The site should have a dedicated page for families to learn more information about hosting a player and players to find host families.
- **Girls** NY Dynamo has a special program for girls teams which should have its own page
- **Tryouts** The tryouts for the upcoming 2026/2027 hockey season are coming up soon, and there should be a dedicated page to learn about the teams and coaches and register to tryout.
- **Store** There should be a dedicated page for the team apparel and gear store.

3. Content & Identity

- As mentioned above, we will want to feature a store for parents and players to buy branded apparel and gear.
- In this version of the site, we are not going to include stats and standings

4. Technical & Administrative Constraints

- For this version, we are not going to protect the site with a login
- This version of the site will not handle payments
- All third party data will be uploaded to the site through a backend system