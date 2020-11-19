# ⚖️  Judging How-To

**Judging is usually one of the most hectic portions of the hackathon. Make sure to read over and be super clear on the judging process.**

### Before event

- Make sure you have a layout of the venue and have a rough idea how you want to lay out the tables for exposition judging.

### During open submission period

- Open devpost and make slack reminders for people to submit when nearing the deadline

### After closing submissions

#### Devpost

- Close Devpost to prevent late submissions
- Export responses from Devpost to get the table assignments
- Exporting Submissions out of Devpost → Get a `csv` of all the project submitted and their data (e.g. who submitted, project description, etc.)
    1. Log into Devpost using `Healthxlogistics@gmail.com`
    2. Go to HealthX under hackathons
    3. Go to `Manage Hackathon > Metrics`
    4. Select `Submission Data`, do not include `Personally Identifiable Information`, and tick `Sort the export by opt-in prize`
    5. Then, hit generate report and download
    6. This will give you a `csv` with these columns

        ```bash
        Opt-in prize
        Submission Title
        Table Number
        Submission Url
        Submission Tagline
        Submission Created At
        Plain Description
        Video
        Website
        File Url
        Built With
        Mlh Points
        If You Submitted For The Domain.Com Prize What Was The Domain You Chose?,
        Mlh Hardware Lab
        Mlh Software Lab
        Submitter Screen Name
        Submitter First Name
        Submitter Last Name
        Submitter Email
        College/Universities Of Team Members
        Additional Team Member Count
        Team Member 1 Screen Name
        Team Member 1 First Name
        Team Member 1 Last Name
        Team Member 1 Email,...
        ```

- Post list of table assignments on Slack (make sure the room layout for judging has tables for all numbers)

#### Table Assignments

- Find list of  judges that have checked in
- Distribute table numbers among judges roughly evenly
- Let each judge know what their table numbers are

#### Expo Judging

- create Google Form to collect judging info
- debrief judges about how judging will work
    - tell judges about form + rubric + password (if needed)
- given total table count, figure out which judges need to visit which tables
- give judges their table list
- let judges roam free
- tally winners using python script or excel functions (decide this before the day of the event!)

#### Sponsor Judging

If a sponsor had decided to judge their sponsor prize with their own team (usually the case) Then we will need to provide them a list of tables to go to. 

- gather a list of which sponsor prizes are being given out
- make sure these are put up on devpost
- gather list of who applied for each sponsor prize using script
- give each sponsor their table list
- make sure to check in with sponsors near end of session to ensure they've picked a winner
- Determine team list for each sponsor
    - Let each sponsor know which tables they should be judging
        - Getting Sponsor Lists → For each Sponsor prize, get a list of tables they need to judge
            1. Download `csv` as per instructions in 'Exporting Submissions out of Devpost'
            2. Rename the `csv` to `submissions.csv`
            3. Make a folder to store scripts and data, with a file hierarchy that looks something like this

                ```python
                scripts/
                    | data/
                    | out/
                    | expo.py # see 'Bad Scripts that jacky made for source'
                    | exportSponsorLists.py # same here
                ```

            4. Place downloaded `csv` into `data/` folder

            5. Run `python exportSponsorLists.py` 

            6. Find the results in `out/`

            If you do not know how / do not want to use the script this can be done through excel as well for a small number of sponsor prizes. 

#### Venue

- Kick people out of main venue to get ready for setup
    - Post message on Slack notifying people to clear out the main space
    - We generally do this in parallel with lunch
- If there is not enough space for the chairs between the aisles fold them and place them under the table
- Tidy up the space (take garbage out, remove all items from table — move to lost and found if necessary)
- Start letting people in once done. Help them find their tables if they need it
- Have some volunteers/organizers help the judges get around if necessary

#### Tallying

- Get all submissions and tally the results to announce finalists. In the case of a tie rejudging may be necessary.

    (link to google form)

    - Expo Judging Score Calculation → Calculate current running average score for each table given current judging form in `csv` format and submissions
        1. Download all the project submissions as `csv` as per instructions in 'Exporting Submissions out of Devpost'
        2. Rename the `csv` to `submissions.csv`
        3. Download the current judging expo form as a `csv` from Google Sheets by doing `File > Download > Comma-separated Values`
        4. Rename the `csv` to `judging.csv`
        5. Run `python expo.py` and the results should be in the terminal, sorted from highest score to lowest score

        ```python
        # e.g.
        Team: TEAMNAME1, Table Number 33 -> Running Average Score: 14.5
        Team: TEAMNAME2, Table Number 5 -> Running Average Score: 12
        Team: TEAMNAME3, Table Number 39 -> Running Average Score: 5.25
        ```

        If you do not know how / do not want to use the script this can be done through excel as well, but may take longer.

- Have a few organizers look through the finalist projects through Github, Devpost, and in-person (if applicable) just to make sure there were no silly mistakes tallying things up, judges using the wrong table number, or just overall caliber or projects.

#### Misc

- Someone to monitor #ask-organizers slack channel and answer submission questions
- Make sure there's time to get lunch between end-of-hacking and judging

### Python Scripts
Export Sponsor Lists

        import csv

        prizes = {}

        with open('data/submissions.csv', newline='\n') as csvfile:
            csvReader = csv.reader(csvfile, dialect='excel')
            next(csvReader)
            for row in csvReader:
                prize = row[0].strip().replace("\xa0", " ").replace("/", ",")

                if prize == "":
                    prize = "none"

                name = row[2]

                if prize not in prizes:
                    prizes[prize] = [name]
                else:
                    prizes[prize].append(name)

        for p in prizes.keys():
            fieldnames = ['team']

            with open('out/%s.csv' % p, 'w', newline='') as file:
                
                writer = csv.DictWriter(file, fieldnames=fieldnames)
                writer.writeheader()
                for team in prizes[p]:
                    writer.writerow({'team': team})

Calculate current averages

        import csv

        teams_num_oc = {}
        teams = {}

        with open('data/judging.csv', newline='\n') as csvfile:
            csvReader = csv.reader(csvfile, dialect='excel')
            next(csvReader)
            for row in csvReader:
                tableNum = row[1]

                if row[7] == '':
                    break

                if tableNum not in teams_num_oc:
                    teams_num_oc[tableNum] = 1
                else:
                    teams_num_oc[tableNum] += 1

                total = int(row[7])

                if tableNum not in teams:
                    teams[tableNum] = total
                else:
                    teams[tableNum] += total

        avged_teams = {}
        for team in teams.keys():
            avged_teams[team] = float(teams[team]) / float(teams_num_oc[team])

        # map from table to name
        team_names = {}
        with open('data/submissions.csv', newline='\n') as csvfile:
            csvReader = csv.reader(csvfile, dialect='excel')
            next(csvReader)
            for row in csvReader:
                team_name = row[1]
                team_number = row[2]

                team_names[team_name] = team_number

        team_names = {v: k for k, v in team_names.items()}

        for t in sorted(avged_teams.items(), reverse=True, key=lambda item: item[1])[:20]:
            try:
                print("Team: %s, Table Number %s -> Running Average Score: %s" % (team_names[t[0]], t[0], t[1]))
            except:
                print("Couldn't find team with table number %s" % t[0])
