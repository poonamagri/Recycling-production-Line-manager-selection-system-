# Recycling-production-Line-manager-selection-system-
Recycling production Line Manager database design,AI prompting, and a functional dashboard 
recycling-candidate-ranking/
│
├── backend/
│   ├── schema.sql
│   ├── sample_data.sql
│
├── ai-prompts/
│   └── candidate_evaluation_prompts.md
│
├── frontend/
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       └── components/
│           ├── Leaderboard.jsx
│           ├── CandidateCard.jsx
│           └── SkillHeatmap.jsx
│
├── faker/
│   └── generateCandidates.js
│
└── README.mdCREATE DATABASE recycling_hr;
USE recycling_hr;

-- Candidates table
CREATE TABLE candidates (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    experience_years INT,
    skills TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- AI evaluations
CREATE TABLE evaluations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    candidate_id INT,
    crisis_management INT CHECK (crisis_management BETWEEN 1 AND 10),
    sustainability INT CHECK (sustainability BETWEEN 1 AND 10),
    team_motivation INT CHECK (team_motivation BETWEEN 1 AND 10),
    FOREIGN KEY (candidate_id) REFERENCES candidates(id)
);

-- Rankings table
CREATE TABLE rankings (
    candidate_id INT PRIMARY KEY,
    total_score INT,
    rank_position INT
);DELIMITER $$

CREATE TRIGGER update_rankings AFTER INSERT ON evaluations
FOR EACH ROW
BEGIN
    REPLACE INTO rankings (candidate_id, total_score)
    SELECT 
        candidate_id,
        (crisis_management + sustainability + team_motivation)
    FROM evaluations
    WHERE candidate_id = NEW.candidate_id;
END$$

DELIMITER ;import { faker } from '@faker-js/faker';
import fs from 'fs';

let sql = '';

for (let i = 0; i < 40; i++) {
  const name = faker.person.fullName();
  const exp = faker.number.int({ min: 2, max: 15 });
  const skills = faker.helpers.arrayElements(
    ['Lean Manufacturing', 'Safety', 'Sustainability', 'Team Leadership', 'Crisis Handling'],
    3
  ).join(', ');

  sql += `INSERT INTO candidates (name, experience_years, skills)
          VALUES ("${name}", ${exp}, "${skills}");\n`;
}

fs.writeFileSync('backend/sample_data.sql', sql);
console.log('40 candidates generated!');## Crisis Management Prompt
Evaluate the candidate’s ability to handle machinery failure, labor shortages,
and safety incidents in a recycling production line.
Score from 1 to 10.

## Sustainability Knowledge Prompt
Evaluate knowledge of recycling processes, waste reduction,
energy efficiency, and environmental compliance.
Score from 1 to 10.

## Team Motivation Prompt
Evaluate leadership style, conflict resolution,
and ability to motivate factory teams.
Score from 1 to 10.import { Container, Title } from '@mantine/core';
import Leaderboard from './components/Leaderboard';

export default function App() {
  return (
    <Container>
      <Title order={2} my="md">🏆 Candidate Leaderboard</Title>
      <Leaderboard />
    </Container>
  );
}import { Table } from '@mantine/core';

const data = [
  { name: 'Amit Sharma', score: 27 },
  { name: 'Neha Verma', score: 25 }
];

export default function Leaderboard() {
  return (
    <Table striped>
      <thead>
        <tr>
          <th>Name</th>
          <th>Total Score</th>
        </tr>
      </thead>
      <tbody>
        {data.map((c, i) => (
          <tr key={i}>
            <td>{c.name}</td>
            <td>{c.score}</td>
          </tr>
        ))}
      </tbody>
    </Table>
  );
}---


