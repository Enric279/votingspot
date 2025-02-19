<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Election Voting System</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }

        /* Navbar Style */
        nav {
            background-color: #4CAF50;
            padding: 10px;
            text-align: center;
        }

        nav a {
            color: white;
            padding: 10px 20px;
            text-decoration: none;
            font-size: 18px;
            margin: 0 10px;
        }

        nav a:hover {
            background-color: #45a049;
        }

        /* Container for Forms */
        .container {
            display: none;
            padding: 20px;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            margin: 20px auto;
            width: 80%;
            max-width: 500px;
        }

        h1 {
            text-align: center;
        }

        label {
            display: block;
            margin-top: 10px;
        }

        input, select, button {
            width: 100%;
            padding: 10px;
            margin-top: 5px;
            font-size: 16px;
        }

        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            margin-top: 10px;
        }

        button:hover {
            background-color: #45a049;
        }

        #message {
            margin-top: 20px;
            text-align: center;
            font-weight: bold;
        }

        #electionsList {
            margin-bottom: 20px;
        }

        .election {
            margin-bottom: 10px;
        }

        .winner {
            font-weight: bold;
            color: green;
        }

    </style>
</head>
<body>

    <!-- Navbar -->
    <nav>
        <a href="#" id="createElectionNav">Create Election</a>
        <a href="#" id="voteElectionNav">Vote in Election</a>
    </nav>

    <!-- Create Election Form -->
    <div class="container" id="createElectionContainer">
        <h1>Create Election</h1>
        <form id="createElectionForm">
            <label for="electionName">Election Name:</label>
            <input type="text" id="electionName" required>
            <label for="candidate1">Candidate 1:</label>
            <input type="text" id="candidate1" required>
            <label for="candidate2">Candidate 2:</label>
            <input type="text" id="candidate2" required>
            <label for="startDate">Start Date:</label>
            <input type="datetime-local" id="startDate" required>
            <label for="endDate">End Date:</label>
            <input type="datetime-local" id="endDate" required>
            <button type="submit">Create Election</button>
        </form>
        <div id="createMessage"></div>
    </div>

    <!-- Vote Election Form -->
    <div class="container" id="voteElectionContainer">
        <h1>Vote in Election</h1>
        <div id="electionsList"></div>
        <form id="voteForm">
            <label for="voterId">Voter ID (4 digits):</label>
            <input type="text" id="voterId" maxlength="4" pattern="\d{4}" required title="Voter ID must be a 4-digit number">
            <label for="voteChoice">Choose Election:</label>
            <select id="voteChoice" required>
                <option value="">Select an Election</option>
            </select>
            <label for="voteCandidate">Choose Candidate:</label>
            <select id="voteCandidate" required>
                <option value="">Select a Candidate</option>
            </select>
            <button type="submit">Vote</button>
        </form>
        <div id="voteMessage"></div>
    </div>

    <script>
        // Store elections in localStorage
        let elections = JSON.parse(localStorage.getItem('elections')) || [];

        // Toggle containers
        function showContainer(containerId) {
            const containers = document.querySelectorAll('.container');
            containers.forEach(container => container.style.display = 'none');
            document.getElementById(containerId).style.display = 'block';
        }

        // Show Create Election form by default
        showContainer('createElectionContainer');

        // Event listeners for navbar links
        document.getElementById('createElectionNav').addEventListener('click', function() {
            showContainer('createElectionContainer');
        });

        document.getElementById('voteElectionNav').addEventListener('click', function() {
            showContainer('voteElectionContainer');
            updateElectionsList();
        });

        // Handle Election Creation
        document.getElementById('createElectionForm').addEventListener('submit', function(e) {
            e.preventDefault();
            const electionName = document.getElementById('electionName').value;
            const candidate1 = document.getElementById('candidate1').value;
            const candidate2 = document.getElementById('candidate2').value;
            const startDate = new Date(document.getElementById('startDate').value);
            const endDate = new Date(document.getElementById('endDate').value);

            const election = {
                name: electionName,
                candidates: [candidate1, candidate2],
                votes: [0, 0], // Initial vote count
                voters: new Set(), // To track who has voted
                startDate: startDate,
                endDate: endDate
            };

            elections.push(election);
            localStorage.setItem('elections', JSON.stringify(elections));

            document.getElementById('createElectionForm').reset();
            document.getElementById('createMessage').textContent = `Election "${electionName}" created successfully!`;
        });

        // Update Elections List for Voting
        function updateElectionsList() {
            const electionsList = document.getElementById('electionsList');
            const voteChoice = document.getElementById('voteChoice');
            electionsList.innerHTML = '';

            // Clear the current election options
            voteChoice.innerHTML = '<option value="">Select an Election</option>';

            elections.forEach((election, index) => {
                // Check if the election is ongoing
                const currentDate = new Date();
                let electionStatus = '';
                if (currentDate < election.startDate) {
                    electionStatus = 'Upcoming';
                } else if (currentDate > election.endDate) {
                    electionStatus = 'Closed';
                    const winnerIndex = election.votes[0] > election.votes[1] ? 0 : 1;
                    electionsList.innerHTML += `<div class="election"><strong>${election.name} (Closed)</strong><br>Winner: ${election.candidates[winnerIndex]}<br><br></div>`;
                } else {
                    electionStatus = 'Ongoing';
                }

                const electionElement = document.createElement('div');
                electionElement.classList.add('election');
                electionElement.innerHTML = `<strong>${election.name}</strong><br>Candidate 1: ${election.candidates[0]} | Candidate 2: ${election.candidates[1]}<br>Status: ${electionStatus}`;
                electionsList.appendChild(electionElement);

                const electionOption = document.createElement('option');
                electionOption.value = index;
                electionOption.textContent = election.name;
                voteChoice.appendChild(electionOption);
            });
        }

        // Update candidate options when an election is selected
        document.getElementById('voteChoice').addEventListener('change', function() {
            const selectedElection = elections[document.getElementById('voteChoice').value];
            const voteCandidate = document.getElementById('voteCandidate');
            voteCandidate.innerHTML = '<option value="">Select a Candidate</option>';

            if (selectedElection && new Date() >= selectedElection.startDate && new Date() <= selectedElection.endDate) {
                selectedElection.candidates.forEach((candidate, index) => {
                    let option = document.createElement('option');
                    option.value = index;
                    option.textContent = candidate;
                    voteCandidate.appendChild(option);
                });
            }
        });

        // Handle Voting
        document.getElementById('voteForm').addEventListener('submit', function(e) {
            e.preventDefault();

            const voterId = document.getElementById('voterId').value;
            const selectedElectionIndex = document.getElementById('voteChoice').value;
            const selectedCandidateIndex = document.getElementById('voteCandidate').value;

            // Validate Voter ID (must be exactly 4 digits)
            const voterIdRegex = /^\d{4}$/;
            if (!voterIdRegex.test(voterId)) {
                document.getElementById('voteMessage').textContent = "Voter ID must be a 4-digit number!";
                return;
            }

            if (!voterId || !selectedElectionIndex || !selectedCandidateIndex) {
                document.getElementById('voteMessage').textContent = "Please fill in all fields.";
                return;
            }

            const election = elections[selectedElectionIndex];

            // Check if the voter has already voted
            if (election.voters.has(voterId)) {
                document.getElementById('voteMessage').textContent = "You have already voted in this election!";
                return;
            }

            // Check if the election is still ongoing
            if (new Date() < election.startDate || new Date() > election.endDate) {
                document.getElementById('voteMessage').textContent = "This election is not currently open for voting.";
                return;
            }

            // Register the vote
            election.voters.add(voterId);
            election.votes[selectedCandidateIndex] += 1;

            // Save updated elections to localStorage
            localStorage.setItem('elections', JSON.stringify(elections));

            document.getElementById('voteMessage').textContent = `Vote registered for "${election.candidates[selectedCandidateIndex]}"!`;
            document.getElementById('voteForm').reset();
        });
    </script>
</body>
</html>
