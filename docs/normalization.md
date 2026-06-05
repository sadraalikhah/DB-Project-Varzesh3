# EER Model Mapping & Normalization to 3NF
## Project: Sports News Website

> **Symbol Guide:**
> - `PK` = Primary Key
> - `FK → Table` = Foreign Key referencing Table
> - `PK/FK` = Both Primary and Foreign Key
> - `NOT NULL` = Required field
> - `UNIQUE` = Must be unique

---


# Part 1: Normalization to 1NF

## Definition
> A relation is in 1NF if it contains no multivalued attributes, no composite attributes, and no nested relations. Every cell must hold exactly one atomic value.

---

## Change 1 — Poll: Remove multivalued attribute `Candidates`

**Functional Dependency violated:** A single Poll tuple contained multiple candidate values, which is not atomic.

**Before (violates 1NF):**
```
Poll(PollID, Status, CreatedAt, UserID, Candidates ← multivalued)
```

**After (1NF):**

Poll:
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PollID | INT | PK | NOT NULL |
| Status | ENUM('Active','Closed') | | NOT NULL |
| CreatedAt | DATETIME | | NOT NULL |
| UserID | INT | FK → User | NOT NULL |

Candidates *(new table — normalizes multivalued attribute)*:
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CandidateID | INT | PK | NOT NULL |
| PollID | INT | FK → Poll | NOT NULL |

**Reason:** `Candidates` was a multivalued attribute on Poll. Each poll can have multiple candidates. Storing them in one field violates 1NF. We extract them into a separate `Candidates` table with a FK back to Poll.

---

## Change 2 — PictureGallery and News: Remove multivalued attribute `Pictures`

**Before (violates 1NF):**
```
PictureGallery(ContentID, Caption, Pictures ← multivalued)
News(ContentID, ArticleText, SecondaryTitle, Pictures ← multivalued)
```

**After (1NF):**

PictureGallery:
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| Caption | TEXT | | |

News:
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| ArticleText | LONGTEXT | | NOT NULL |
| SecondaryTitle | VARCHAR(255) | | |

Picture *(new table — normalizes multivalued attribute from both PictureGallery and News)*:
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PictureID | INT | PK | NOT NULL |
| GalleryID | INT | FK → PictureGallery | nullable |
| NewsID | INT | FK → News | nullable |

**Reason:** Both `PictureGallery` and `News` had a multivalued `Pictures` attribute — each could have many images. Storing multiple URLs in one field violates 1NF. We extract them into a separate `Picture` table. `PictureID` is a new surrogate key added during mapping since the original EER had no key for this multivalued attribute.

---

---

# Part 2: Normalization to 2NF

## Definition
> A relation is in 2NF if it is in 1NF and every non-prime attribute is fully functionally dependent on the entire primary key (no partial dependencies). This only applies to tables with composite keys.

---

## Change — PlayerParticipatesInMatch: Remove partial dependency on `MatchLocation`

**Composite key:** `(PlayerID, MatchID)`

**Before (violates 2NF):**
```
PlayerParticipatesInMatch(PlayerID, MatchID, PerformanceScore, MatchLocation)
```

**Functional dependencies:**
```
(PlayerID, MatchID) → PerformanceScore   ✅ full dependency
MatchID             → MatchLocation      ❌ partial dependency — depends only on part of key
```

`MatchLocation` depends only on `MatchID`, not on the full composite key `(PlayerID, MatchID)`. This is a partial dependency — a violation of 2NF.

**After (2NF):**

PlayerParticipatesInMatch:
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Player | NOT NULL |
| MatchID | INT | PK/FK → Match | NOT NULL |
| PerformanceScore | DECIMAL(5,2) | | |

> `MatchLocation` is removed — it already exists as `Location` in the `Match` table where it belongs.

**Reason:** `MatchLocation` was functionally determined by `MatchID` alone, not by the full key `(PlayerID, MatchID)`. Keeping it here would cause update anomalies (changing a match location would require updating every player row for that match). Moving it to `Match` where it belongs resolves the partial dependency.

---

---

# Part 3: Normalization to 3NF

## Definition
> A relation is in 3NF if it is in 2NF and no non-prime attribute is transitively dependent on the primary key. That is, there is no X → Y → Z where X is the PK and Y is not a candidate key.

---

## Change — Person: Remove transitive dependency on `SportID`

**Before (violates 3NF):**
```
Person(PersonID, FullName, BirthDate, Nationality, Gender, PhotoURL, IsRetired, SportID)
```

**Functional dependencies:**
```
PersonID → SportID        (each person is associated with a sport)
SportID  → Sport.Name     (each sport has a name)
```

Therefore:
```
PersonID → SportID → Sport.Name
```

This is a **transitive dependency**: `PersonID → SportID → SportName`. Since `SportID` is not a candidate key of `Person`, this violates 3NF.

Furthermore, a player's sport can already be derived through:
```
Player → PlayerTeam → Team → Sport
```

Storing `SportID` directly in `Person` creates redundancy.

**After (3NF):**

Person:
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PersonID | INT | PK | NOT NULL |
| FullName | VARCHAR(150) | | NOT NULL |
| BirthDate | DATE | | |
| Nationality | VARCHAR(100) | | |
| Gender | ENUM('Male','Female','Other') | | |
| PhotoURL | VARCHAR(255) | | |
| IsRetired | BOOLEAN | | DEFAULT FALSE |

> `SportID` removed. A person's sport is derivable via `Player → PlayerTeam → Team → Competition → Sport`.

**Reason:** `SportID` in `Person` created a transitive dependency `PersonID → SportID → SportName`. Since `SportID` is not a candidate key, this violates 3NF and causes update anomalies (if a sport's data changes, every Person row referencing it must be updated). Removing it and relying on the existing Team–Sport relationship eliminates this redundancy.

---

---

# Part 4: Final Schema (After 3NF)

## 1. Sports Structure Subsystem

### Sport
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| SportID | INT | PK | NOT NULL |
| Name | VARCHAR(100) | | NOT NULL |

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

### League *(Subclass of Competition — Disjoint)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CompetitionID | INT | PK/FK → Competition | NOT NULL |
| Week | INT | | |
| HomeAwayFormat | BOOLEAN | | |
| NumberOfTeams | INT | | |

### Cup *(Subclass of Competition — Disjoint)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CompetitionID | INT | PK/FK → Competition | NOT NULL |

### Tournament *(Subclass of Competition — Disjoint)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CompetitionID | INT | PK/FK → Competition | NOT NULL |
| HostCountry | VARCHAR(100) | | |
| GroupCount | INT | | |

### CompetitionGroup
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| GroupID | INT | PK | NOT NULL |
| GroupName | VARCHAR(100) | | NOT NULL |
| CompetitionID | INT | FK → Competition | NOT NULL |

### Team
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| TeamID | INT | PK | NOT NULL |
| TeamName | VARCHAR(100) | | NOT NULL |
| Logo | VARCHAR(255) | | |
| FoundedYear | YEAR | | |
| Type | ENUM('Club','National') | | NOT NULL |
| Location | VARCHAR(255) | | |

### GroupTeam *(Junction Table — M:N between CompetitionGroup and Team)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| GroupID | INT | PK/FK → CompetitionGroup | NOT NULL |
| TeamID | INT | PK/FK → Team | NOT NULL |

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

### MatchTeam *(Junction Table — exactly 2 rows per match)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchID | INT | PK/FK → Match | NOT NULL |
| TeamID | INT | PK/FK → Team | NOT NULL |
| Role | ENUM('Home','Away') |  PK | NOT NULL |
| Score | INT | | |

### Formation *(1:1 with Match)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchID | INT | PK/FK → Match | NOT NULL |
| HomeFormationType | VARCHAR(20) | | |
| AwayFormationType | VARCHAR(20) | | |

### PlayerFormation *(Junction Table)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Player | NOT NULL |
| MatchID | INT | PK/FK → Match | NOT NULL |

### MatchEvent
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchEventID | INT | PK | NOT NULL |
| Minute | INT | | NOT NULL |
| Type | ENUM('Goal','YellowCard','RedCard','Substitution') | | NOT NULL |
| MatchID | INT | FK → Match | NOT NULL |
| PlayerID | INT | FK → Player | |

### MatchVideos *(Junction Table)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| MatchID | INT | PK/FK → Match | NOT NULL |
| VideoID | INT | PK/FK → Video | NOT NULL |

### PlayerTeam *(Junction Table — M:N between Player and Team)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Player | NOT NULL |
| TeamID | INT | PK/FK → Team | NOT NULL |
| StartDate | DATE | PK | NOT NULL |
| EndDate | DATE | | |

### PlayerParticipatesInMatch *(After 2NF — MatchLocation removed)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Player | NOT NULL |
| MatchID | INT | PK/FK → Match | NOT NULL |
| PerformanceScore | DECIMAL(5,2) | | |

---

## 2. People Management Subsystem

### Person *(After 3NF — SportID removed)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PersonID | INT | PK | NOT NULL |
| FullName | VARCHAR(150) | | NOT NULL |
| BirthDate | DATE | | |
| Nationality | VARCHAR(100) | | |
| Gender | ENUM('Male','Female','Other') | | |
| PhotoURL | VARCHAR(255) | | |
| IsRetired | BOOLEAN | | DEFAULT FALSE |

### Player *(Subclass of Person — Overlapping)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PlayerID | INT | PK/FK → Person | NOT NULL |
| Position | VARCHAR(50) | | |
| JerseyNumber | INT | | |
| MarketValue | DECIMAL(15,2) | | |

### Coach *(Subclass of Person — Overlapping)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CoachID | INT | PK/FK → Person | NOT NULL |
| CoachType | ENUM('Head','Assistant','Goalkeeping') | | |
| TeamID | INT | FK → Team | NOT NULL |

### Publisher
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PublisherID | INT | PK | NOT NULL |
| Name | VARCHAR(150) | | NOT NULL |
| Email | VARCHAR(255) | | UNIQUE |
| Role | ENUM('Editor','Journalist','Reporter') | | |

### Photographer *(Subclass of Publisher)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PublisherID | INT | PK/FK → Publisher | NOT NULL |
| PublicProfilePhoto | VARCHAR(255) | | |

---

## 3. Content Management Subsystem

### Content
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK | NOT NULL |
| Title | VARCHAR(255) | | NOT NULL |
| PublishDate | DATETIME | | |
| CreatedAt | DATETIME | | NOT NULL |
| ThumbnailURL | VARCHAR(255) | | |
| LikeCount | INT | | DEFAULT 0 |
| ViewCount | INT | | DEFAULT 0 |
| SportID | INT | FK → Sport | |
| PublisherID | INT | FK → Publisher | NOT NULL |

### News *(Subclass of Content — Disjoint)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| ArticleText | LONGTEXT | | NOT NULL |
| SecondaryTitle | VARCHAR(255) | | |

### Story *(Subclass of Content — Disjoint)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| VideoURL | VARCHAR(255) | | |

### Video *(Subclass of Content — Disjoint)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| VideoURL | VARCHAR(255) | | NOT NULL |
| Duration | INT | | |
| Category | VARCHAR(100) | | |
| Description | TEXT | | |

### PictureGallery *(Subclass of Content — Disjoint, After 1NF)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| Caption | TEXT | | |

### Picture *(After 1NF — normalizes multivalued attribute from PictureGallery and News)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PictureID | INT | PK | NOT NULL |
| GalleryID | INT | FK → PictureGallery | nullable |
| NewsID | INT | FK → News | nullable |

### Tag
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| TagID | INT | PK | NOT NULL |
| Name | VARCHAR(100) | | NOT NULL, UNIQUE |

### ContentTag *(Junction Table)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| TagID | INT | PK/FK → Tag | NOT NULL |

### TeamAppearsInContent
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| TeamID | INT | PK/FK → Team | NOT NULL |
| ContentID | INT | PK/FK → Content | NOT NULL |

### CompetitionMentionedInContent
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| ContentID | INT | PK/FK → Content | NOT NULL |
| CompetitionID | INT | PK/FK → Competition | NOT NULL |

### PersonMentionedInContent
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PersonID | INT | PK/FK → Person | NOT NULL |
| ContentID | INT | PK/FK → Content | NOT NULL |

---

## 4. User Interaction Subsystem

### User
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| UserID | INT | PK | NOT NULL |
| Username | VARCHAR(100) | | NOT NULL, UNIQUE |
| PasswordHash | VARCHAR(255) | | NOT NULL |
| JoinDate | DATE | | NOT NULL |
| PhoneNumber | VARCHAR(20) | | |
| ProfilePhotoURL | VARCHAR(255) | | |

### UserViewsContent
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| UserID | INT | PK/FK → User | NOT NULL |
| ContentID | INT | PK/FK → Content | NOT NULL |

### Comment
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CommentID | INT | PK | NOT NULL |
| Text | TEXT | | NOT NULL |
| CommentDate | DATETIME | | NOT NULL |
| LikeCount | INT | | DEFAULT 0 |
| DislikeCount | INT | | DEFAULT 0 |
| UserID | INT | FK → User | NOT NULL |
| ContentID | INT | FK → Content | NOT NULL |
| ParentCommentID | INT | FK → Comment | nullable |

### Like
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| LikeID | INT | PK | NOT NULL |
| UserID | INT | FK → User | NOT NULL |
| ContentID | INT | FK → Content | nullable |

### Poll *(After 1NF — Candidates removed)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PollID | INT | PK | NOT NULL |
| Status | ENUM('Active','Closed') | | NOT NULL |
| CreatedAt | DATETIME | | NOT NULL |
| UserID | INT | FK → User | NOT NULL |

### Candidates *(After 1NF — normalizes multivalued attribute from Poll)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| CandidateID | INT | PK | NOT NULL |
| PollID | INT | FK → Poll | NOT NULL |

### PollOption

| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| OptionID | INT | PK | NOT NULL |
| OptionText | VARCHAR(255) | | NOT NULL |
| PollID | INT | FK → Poll | NOT NULL |

### UserVotesInPoll *(Junction Table)*
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PollID | INT | PK/FK → Poll | NOT NULL |
| UserID | INT | PK/FK → User | NOT NULL |
| UserVote | VARCHAR(255) | | |

---

## 5. Prediction System Subsystem

### Prediction
| Column | Type | Key | Constraint |
|--------|------|-----|------------|
| PredictionID | INT | PK | NOT NULL |
| Result | VARCHAR(100) | | NOT NULL |
| PredictionTime | DATETIME | | NOT NULL |
| MatchID | INT | FK → Match | NOT NULL |
| UserID | INT | FK → User | NOT NULL |

---

---

# Normalization Summary

| Step | Table | Change | Reason |
|------|-------|--------|--------|
| **1NF** | Poll | Extracted `Candidates` into separate `Candidates(CandidateID, PollID)` table | `Candidates` was a multivalued attribute — each poll had multiple candidate values in one field |
| **1NF** | PictureGallery / News | Extracted `Pictures` into separate `Picture(PictureID, GalleryID, NewsID)` table | `Pictures` was a multivalued attribute on both entities |
| **2NF** | PlayerParticipatesInMatch | Removed `MatchLocation` column | `MatchLocation` depended only on `MatchID` (partial dependency on composite key) — it already exists in `Match.Location` |
| **3NF** | Person | Removed `SportID` column | Transitive dependency: `PersonID → SportID → SportName`. Sport is derivable via `Player → PlayerTeam → Team → Sport` |

