# EER Model Mapping to Relational Schema

> **Symbol Guide:**
> - `PK` = Primary Key
> - `FK → Table` = Foreign Key referencing Table
> - `PK/FK` = Both Primary and Foreign Key
> - `NOT NULL` = Required field
> - `UNIQUE` = Must be unique

---

## 1. Sports Structure Subsystem

---

### Sport

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| SportID | INT | PK | NOT NULL |
| Name | VARCHAR(100) | | NOT NULL |

---

### Competition

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CompetitionID | INT | PK | NOT NULL |
| Name | VARCHAR(100) | | NOT NULL |
| Country | VARCHAR(100) | | |
| StartDate | DATE | | |
| EndDate | DATE | | |
| LogoURL | VARCHAR(255) | | |
| SportID | INT | FK → Sport | NOT NULL |

---

### League *(Subclass of Competition — Disjoint)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CompetitionID | INT | PK/FK → Competition | NOT NULL |
| Week | INT | | |
| HomeAwayFormat | BOOLEAN | | |
| NumberOfTeams | INT | | |

---

### Cup *(Subclass of Competition — Disjoint)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CompetitionID | INT | PK/FK → Competition | NOT NULL |

---

### Tournament *(Subclass of Competition — Disjoint)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CompetitionID | INT | PK/FK → Competition | NOT NULL |
| HostCountry | VARCHAR(100) | | |
| GroupCount | INT | | |

---

### CompetitionGroup

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| GroupID | INT | PK | NOT NULL |
| GroupName | VARCHAR(100) | | NOT NULL |
| CompetitionID | INT | FK → Competition | NOT NULL |

---

### Team

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| TeamID | INT | PK | NOT NULL |
| TeamName | VARCHAR(100) | | NOT NULL |
| Logo | VARCHAR(255) | | |
| FoundedYear | YEAR | | |
| Type | ENUM('Club','National') | | NOT NULL |
| Location | VARCHAR(255) | | |

---

### GroupTeam *(Junction Table — M:N between CompetitionGroup and Team)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| GroupID | INT | PK/FK → CompetitionGroup | NOT NULL |
| TeamID | INT | PK/FK → Team | NOT NULL |

---

### Match

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchID | INT | PK | NOT NULL |
| MatchDate | DATETIME | | NOT NULL |
| MatchStatus | ENUM('Scheduled','Live','Finished') | | NOT NULL |
| Location | VARCHAR(255) | | |
| Referee | VARCHAR(100) | | |
| FullMatchURL | VARCHAR(255) | | |
| Stats | TEXT | | |
| CompetitionID | INT | FK → Competition | NOT NULL |

---

### MatchTeam *(Junction Table — M:N between Match and Team, exactly 2 teams per match)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchID | INT | PK/FK → Match | NOT NULL |
| TeamID | INT | PK/FK → Team | NOT NULL |
| Role | ENUM('Home','Away') | PK | NOT NULL |
| Score | INT | | |

---

### Formation

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchID | INT | PK/FK → Match | NOT NULL |
| HomeFormationType | VARCHAR(20) | | |
| AwayFormationType | VARCHAR(20) | | |

---

### PlayerFormation *(Junction Table — Player is in Formation, relationship: player is in formation)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Player | NOT NULL |
| MatchID | INT | PK/FK → Match | NOT NULL |

---

### MatchEvent

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchEventID | INT | PK | NOT NULL |
| Minute | INT | | NOT NULL |
| Type | ENUM('Goal','YellowCard','RedCard','Substitution') | | NOT NULL |
| MatchID | INT | FK → Match | NOT NULL |
| PlayerID | INT | FK → Player | |

---

### MatchVideos *(Junction Table — Match has Videos)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchID | INT | PK/FK → Match | NOT NULL |
| VideoID | INT | PK/FK → Video | NOT NULL |

---

### PlayerTeam*(Junction Table — M:N between Player and Team with attributes)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Player | NOT NULL |
| TeamID | INT | PK/FK → Team | NOT NULL |


---

### PlayerParticipatesInMatch *(Junction Table — Player participates in Match)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Player | NOT NULL |
| MatchID | INT | PK/FK → Match | NOT NULL |
| PerformanceScore | DECIMAL(5,2) | | |

---

## 2. People Management Subsystem

---

### Person

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PersonID | INT | PK | NOT NULL |
| FullName | VARCHAR(150) | | NOT NULL |
| BirthDate | DATE | | |
| Nationality | VARCHAR(100) | | |
| Gender | ENUM('Male','Female','Other') | | |
| PhotoURL | VARCHAR(255) | | |
| IsRetired | BOOLEAN | | DEFAULT FALSE |
| SportID | INT | PK/FK → Sport | NOT NULL |

---

### Player *(Subclass of Person — Overlapping)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Person | NOT NULL |
| Position | VARCHAR(50) | | |
| JerseyNumber | INT | | |
| MarketValue | DECIMAL(15,2) | | |

---

### Coach *(Subclass of Person — Overlapping)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CoachID | INT | PK/FK → Person | NOT NULL |
| CoachType | ENUM('Head','Assistant','Goalkeeping') | | |
| TeamID | INT | FK → Team | NOT NULL |

---

### Publisher

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PublisherID | INT | PK | NOT NULL |
| Name | VARCHAR(150) | | NOT NULL |
| Email | VARCHAR(255) | | UNIQUE |
| Role | ENUM('Editor','Journalist','Reporter') | | |

---

### Photographer *(Subclass of Publisher)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PublisherID | INT | PK/FK → Publisher | NOT NULL |
| PublicProfilePhoto | VARCHAR(255) | | |

---

## 3. Content Management Subsystem

---

### Content

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK | NOT NULL |
| Title | VARCHAR(255) | | NOT NULL |
| PublishDate | DATETIME | | |
| CreatedAt | DATETIME | | NOT NULL |
| ThumbnailURL | VARCHAR(255) | | |
| SportID | INT | FK → Sport | |


---

### News *(Subclass of Content — Disjoint)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| ArticleText | LONGTEXT | | NOT NULL |
| SecondaryTitle | VARCHAR(255) | | |

---

### Story *(Subclass of Content — Disjoint)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| VideoURL | VARCHAR(255) | | |

---

### Video *(Subclass of Content — Disjoint)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| VideoURL | VARCHAR(255) | | NOT NULL |
| Duration | INT | | |
| Category | VARCHAR(100) | | |
| Description | TEXT | | |

---

### PictureGallery *(Subclass of Content — Disjoint)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PictureID | INT | PK | NOT NULL |
| GalleryID | INT | FK → PictureGallery | |
| NewsID | INT | FK → News | |

---

### Picture *(normalizes multivalued attribute Pictures)*



### Tag

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| TagID | INT | PK | NOT NULL |
| Name | VARCHAR(100) | | NOT NULL, UNIQUE |

---

### ContentTag *(Junction Table — M:N between Content and Tag)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| TagID | INT | PK/FK → Tag | NOT NULL |

---

### TeamAppearsInContent *(Junction Table — Team appears_in Content)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| TeamID | INT | PK/FK → Team | NOT NULL |
| ContentID | INT | PK/FK → Content | NOT NULL |

---

### CompetitionMentionedInContent *(Junction Table — Competition mentioned_in Content)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| CompetitionID | INT | PK/FK → Competition | NOT NULL |

---

### PersonMentionedInContent *(Junction Table — Person mentioned_in Content)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PersonID | INT | PK/FK → Person | NOT NULL |
| ContentID | INT | PK/FK → Content | NOT NULL |

---

### PublisherCreatesContent *(relationship: Publisher creates Content)*
 
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PublisherID | INT | PK/FK → Publisher | NOT NULL |
| ContentID | INT | PK/FK → Content | NOT NULL |

---

## 4. User Interaction Subsystem

---

### User

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| UserID | INT | PK | NOT NULL |
| Username | VARCHAR(100) | | NOT NULL, UNIQUE |
| PasswordHash | VARCHAR(255) | | NOT NULL |
| JoinDate | DATE | | NOT NULL |
| PhoneNumber | VARCHAR(20) | | |
| ProfilePhotoURL | VARCHAR(255) | | |

---

### UserViewsContent *(Junction Table — User views Content)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| UserID | INT | PK/FK → User | NOT NULL |
| ContentID | INT | PK/FK → Content | NOT NULL |


---

### Comment

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CommentID | INT | PK | NOT NULL |
| Text | TEXT | | NOT NULL |
| CommentDate | DATETIME | | NOT NULL |
| UserID | INT | FK → User | NOT NULL |
| ContentID | INT | FK → Content | NOT NULL |
| ParentCommentID | INT | FK → Comment | nullable (for replies) |

---

### Like *(Junction Table — User likes Content or Comment)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| LikeID | INT | PK | NOT NULL |
| UserID | INT | FK → User | NOT NULL |
| ContentID | INT | FK → Content | nullable |
| CommentID | INT | FK → Comment | nullable |

---

### Poll

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PollID | INT | PK | NOT NULL |
| Status | ENUM('Active','Closed') | | NOT NULL |


---

### PollOption *(normalizes multivalued attribute 'Candidates' from Poll)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| OptionID | INT | PK | NOT NULL |
| OptionText | VARCHAR(255) | | NOT NULL |
| PollID | INT | FK → Poll | NOT NULL |

---

### Candidates
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CandidatesID | INT | PK | NOT NULL |
| PollID | INT | FK → Poll | NOT NULL |

---

### UserVotesInPoll *(Junction Table — User votes_in Poll)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PollID | INT | PK/FK → Poll | NOT NULL |
| UserID | INT | PK/FK → User | NOT NULL |
| UserVote | VARCHAR(255) | | |

---

### UserWritesComment *(Junction Table — User writes Comment)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CommentID | INT | PK/FK → Comment | NOT NULL |
| UserID | INT | PK/FK → User | NOT NULL |


## 5. Prediction System Subsystem

---

### Prediction

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PredictionID | INT | PK | NOT NULL |
| Result | VARCHAR(100) | | NOT NULL |
| MatchID | INT | FK → Match | NOT NULL |


---

### UserMakesPrediction *(relationship: User makes Prediction)*

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PredictionID | INT | PK/FK → PolPrediction | NOT NULL |
| UserID | INT | PK/FK → User | NOT NULL |
| PredictionTime | DATETIME | | |


---

## Table Summary

| # | Table Name | Type | Subsystem |
|---|------------|------|-----------|
| 1 | Sport | Strong Entity | Sports Structure |
| 2 | Competition | Strong Entity (Parent) | Sports Structure |
| 3 | League | Disjoint Subclass | Sports Structure |
| 4 | Cup | Disjoint Subclass | Sports Structure |
| 5 | Tournament | Disjoint Subclass | Sports Structure |
| 6 | CompetitionGroup | Strong Entity | Sports Structure |
| 7 | GroupTeam | M:N Junction Table | Sports Structure |
| 8 | Team | Strong Entity | Sports Structure |
| 9 | Match | Strong Entity | Sports Structure |
| 10 | MatchTeam | M:N Junction Table (2 rows per match) | Sports Structure |
| 11 | Formation | Weak Entity (1:1 with Match) | Sports Structure |
| 12 | PlayerFormation | M:N Junction Table | Sports Structure |
| 13 | MatchEvent | Weak Entity | Sports Structure |
| 14 | MatchVideos | M:N Junction Table | Sports Structure |
| 15 | PlayerTeamHistory | M:N Junction Table with attributes | Sports Structure |
| 16 | PlayerParticipatesInMatch | M:N Junction Table | Sports Structure |
| 17 | Person | Strong Entity (Parent) | People Management |
| 18 | Player | Overlapping Subclass | People Management |
| 19 | Coach | Overlapping Subclass | People Management |
| 20 | Publisher | Strong Entity | People Management |
| 21 | Photographer | Subclass of Publisher | People Management |
| 22 | Content | Strong Entity (Parent) | Content Management |
| 23 | News | Disjoint Subclass | Content Management |
| 24 | Story | Disjoint Subclass | Content Management |
| 25 | Video | Disjoint Subclass | Content Management |
| 26 | PictureGallery | Disjoint Subclass | Content Management |
| 27 | Picture | Weak Entity (normalizes multivalued attr) | Content Management |
| 28 | Tag | Strong Entity | Content Management |
| 29 | ContentTag | M:N Junction Table | Content Management |
| 30 | TeamAppearsInContent | M:N Junction Table | Content Management |
| 31 | CompetitionMentionedInContent | M:N Junction Table | Content Management |
| 32 | PersonMentionedInContent | M:N Junction Table | Content Management |
| 33 | User | Strong Entity | User Interaction |
| 34 | UserViewsContent | M:N Junction Table | User Interaction |
| 35 | Comment | Strong Entity (Recursive) | User Interaction |
| 36 | Like | Junction Table (Content or Comment) | User Interaction |
| 37 | Poll | Strong Entity | User Interaction |
| 38 | PollOption | Weak Entity (normalizes Candidates) | User Interaction |
| 39 | UserVotesInPoll | M:N Junction Table | User Interaction |
| 40 | Prediction | Strong Entity | Prediction System |
