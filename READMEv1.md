# gmail2markdown
Emails from gmail to markdown. Used for Obsidian, AI digestion, etc..

- Works according to unified module blueprint
- primary use case: inside gmail, filtering settings are set up to label emails matching certain criteria
- this ool is explicityly looking for emails with a particular label, muct like how the reddit scraper looks in specific subreddits
- instead of min_post_age, we are more concerned with max_post_age so we don't pull last year's emails.
- we should also have a method of addressing "there's an email in this label that's past the min_post_age for this scrape, but it hasn't ever been scraped and it's still unread." once this is up and running periodically pulling from gmail, i'm sure it will be getting everything, and the max_post_age can be lenient. if you aren't sure about if you've seen all the emails, you probably aren't limiting it to 24 hours. default can be something like 5 days. similarly, might be worth having high defaults for max amount of posts, or just don't set it by default, it's useful for debug though.
- in the sandman project, in MVP stage, these emails are gettings stored, then immediately there's a second automation that scrapes from these emails. this is becuase we are (in MVP) specifically looking for job alert emails. i have signed up for alerts from indeed and linkedin. I have certain search terms where i get periodic alerts emailed to me. we also must capture that info (what was the query?) as best as we can.
- in these emails there will be a list of jobs to consider applying to. so the secondary automation will be to create a markdown file for each of these jobs, capturing the URL. Acceptance criteria for full MVP state is that we have scraped emails, they have keyIDs, then in each email, we scraped each job, which also have KeyIDs, they also have the ID of the email they came from and the querie from that email. essentially two DBs that have relation fields. one email to many jobs. one job to one email. etc.
- acceptance criteria for full MVP state: the email only gives limited info for each job: title, location, URL, etc. they rarely even have a summary, let along the full JD. either we scrape the full content as soon as we get the URL, or after gmail scraper is built we need another scraper to handle linedin, one to handle indeed, one to handle other sites. etc. My instinct is that there's no need to build those. they must be on github already. There's probably dependencies i can use for "universal scraper" and point it at the job listing URL, OR i can find a linkedin scraper and a indeed scraper and so on, and use them all in my "job scraper". which can scrape from thes emails and ALSO just connect direct to the website to monitor listings according to our specific queries. MVP might just concern itself with the emails, the face that they are from linkedin or indeed or whatever, that might be out of scope. the task of extracting job links from the emails should be in scope, but maybe disabled by default for most users.

note on the universal scraper or job scraper: we also need to support manual entry from user. in current version of job script, there's a apple shortcut that user can use to send the job detail from current page and send it to the server to parse. this feature is a must, but i'm not sure who's scope it falls under. if there are three services we couuld have:
	- email scraper
	- universal wep page scraper (something like beautiful soup or something)
	- job scraper (can scrape from the emails and turn it into our "job" format markdown, can scrape from the manual input from user, can scrape from universal web scraper)

that would open up chains like 
1. 4 job alert emails scraped, and they have the front-matter showing "label: job-alert" but since they don't have front-matter field "jobs-scraped", on the next cycle of the jobs scraper, it will see it's in quue, it scrapes jobs from those emails, labels each email "jobs-scraped: 6" or whatever. since it doesn't have content, only really has front-matter, we can either call universal scraper, use jobscraper connection to site, or infer it hasn't gotten a full scrape, it's now in queue for one of those two processes on thier next cycle.
eventually we end up with full email scraped, partial jobs from each email, then they turn into full jobs with feull detail after full scrape.


initial research to do: once we have the job titles and URLs, how feasible is it to just scrape from that URL? In my current script, i have no way to automate that full-page scrape. only can be done via a manual trigger using apple shortcuts.
full logic and working functional version of this was made in appscript, can be found here: ``/docs/_organized chaos/scraping job links from emails``

I'm thinking there must be a few modules, or they're combined into a larger module or something
- scrape email into markdown
- extract jobs and create the database of jobs (from emails, but also needs to be able to be from manual trigger on a web page)
- method of querying ljob boards with specific search logic (APIs?)
	- linkedin
	- indeed
	- others 
- general web scraping utility that takes a URL and returns the JSON/markdown.


I can imagine the job scraper will be autoamtivcally called as a nececary last step any time there's a call from this gmail scraper that results in job alerts being scraped. each link in that email turns into a abreviated job page, but might only have enough info for some front-matter, no actual content.

The integrations with sites like linkedin, indeed, and others will be it's own step. I imagine there are several solutions available on github right now that use python to monitor job posts acros several platforms. let's try to use that instead of rebuilding it all. hopefully that will also let us get the full job content, so we don't need to create and fine-tune a scraper utility for each source of job posts. When it comes to less-used sources, we will probably end up needing something to scrape it into a markdown file, something like beautifulsoup, markitdown, etc. I don't want to totally rebuild this same tool every time i make a module.


some considerations about this module "gmail2markdown"
- there is no actual requirement that we use the gmail API, it's just the first diection i thought to go.
- it is very simple for me to set up in gmail "everything that is a job alert, forward to this other email address", and we can intake emails that way
- similarly, i can very easily hook my gmail up to a email client, and (using apple's mail app as an example) gmail organizes emails into folders bsed on their label. we could hook up the gmail addres to a minimal self-hosted email client or something on rascal (the pi) or on the mackbook, and only sync that folder. each new email triggering an automation according it the sender email address.


---


# 4 questions:

   1. rather than re-invent the wheel for each scraper module. we can look at
   this simple process. User has a URL, user needs the page at that URL
   scraped, the resulting HTML/json is then optimized, and we store a json file
   that's very minimal, only storing the information we actually care about.
   from that, we make a markdown file according to the user's prefernces in
   their markdown template. What dependencies would make sense here, so i'm not
   re-writing over and over? I've heard of things like markitdown,
   beautifylsoup, etc... i don't know much about scraping, but i am willing to
   totally bypass the json part of the equation if it's simpler to just go from
   html page scrape to a markdown file that we can then just refine down to
   what we want according to our template. What tools exist that would make all
   this much more predictable and simple? I have no real way to scrape pages
   from URL, other than weird workarounds like how i use the rss feed from
   reddit to get json files.

   2. this "sandman" orchestration layer seems like something that doesn't need
   to be re-invented. there are things like n8n and i just found one on gihub
   that looks great it's called paperclip at
   https://github.com/paperclipai/paperclip what might work for this project,
   so i can just focus on modules instead of the lagic of the orchestration
   layer that needs all sorts of UX polish. What options exist for "sandman" so
   i am not re-writing it all myself.

   3. aside from taking a web page and scraping it to markdown, as we talked
   about in #1, what dependencies or APIs or other resources exist that we
   could use to monitor job sites with specifc search criteria? is there
   linkedIn API and indeed API and all those other websites? is there a good
   open source python resource that let's us monitor them all with multiple
   search queries and filters? are we just going to need a list of websites,
   each with their own API, and test out with trial and error different queries
   over time to see what kind of results we get? is it mostly going to just be
   something we build ourselves, or is there already "aggregator" feeds that
   can be queried via api or something? Ultimately, linkedin and indeed tend to
   be bloated with fake jobs and at the same time, very competitive for every
   job posted there. I'd rather not spend more than 50% of the effort on those,
   and make sure at least 50% of my effort is setting up feeds from less
   conventional, less competitive, less congested sources.

   4. Take a look at the begining notes to start off the readme file for
   gmail2markdown (the new module for scraping emails). In there i discuss the
   vision for how the email scraper will be used and how it will feed into the
   rest of this ecosystem, and how it's just used as one more source of job
   listings. this might not be a priority if we can do periodic queries and
   monitor results over time, but it's a low-tech solution for right now. More
   importantly, those rambling notes should give enough info for you to start
   making a real readme file and a real architecture document for the email
   scraper, and it gives plenty of context about what's needed for job scraper,
   manual input from client, and other related services that need to work in
   this project. Help me organize all this, so we have clear direction on where
   to go. most importantly, what dependencies would give us the most
   straight-forward way to accomplish the kind of scraping automations I'm
   looking to do here? review readmev1.md in gmail2markdown module, create a
   readmev2, and use that file to address all these four points

