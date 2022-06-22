---
tags: [Mermaid]
mermaid: true
---

# Splinter v0.7 Database Design

## Scabbard v3

<div class="mermaid">
erDiagram
    scabbard_service {
        Text service_id PK
        Text status
        Text consensus
    }

    scabbard_alarm {
        Text service_id PK
        Text alarm_type PK
        BigInt alarm
    }

    scabbard_alarm }o--|| scabbard_service: contains

    scabbard_peer {
        Text service_id PK
        Text peer_service_id PK
    }

    scabbard_peer }o--|| scabbard_service: contains
</div>

### scabbard_alarm

#### Diesel

```rust
table! {
    scabbard_alarm (service_id, alarm_type) {
        service_id -> Text,
        alarm_type -> Text,
        alarm -> BigInt,
    }
}
```

#### PostgreSQL

```
              Table "public.scabbard_alarm"
   Column   |    Type    | Collation | Nullable | Default
------------+------------+-----------+----------+---------
 service_id | text       |           | not null |
 alarm_type | alarm_type |           | not null |
 alarm      | bigint     |           | not null |
Indexes:
    "scabbard_alarm_pkey" PRIMARY KEY, btree (service_id, alarm_type)
Foreign-key constraints:
    "scabbard_alarm_service_id_fkey" FOREIGN KEY (service_id) REFERENCES scabbard_service(service_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE scabbard_alarm (
    service_id                TEXT NOT NULL,
    alarm_type                TEXT NOT NULL
    CHECK ( alarm_type IN ('TWOPHASECOMMIT')),
    alarm                     BIGINT NOT NULL,
    FOREIGN KEY (service_id) REFERENCES scabbard_service(service_id) ON DELETE CASCADE,
    PRIMARY KEY (service_id, alarm_type)
);
```

### scabbard_peer

#### Diesel

```rust
table! {
    scabbard_peer (service_id, peer_service_id) {
        service_id  -> Text,
        peer_service_id  -> Text,
    }
}
```

#### PostgreSQL

```
              Table "public.scabbard_peer"
     Column      | Type | Collation | Nullable | Default
-----------------+------+-----------+----------+---------
 service_id      | text |           | not null |
 peer_service_id | text |           | not null |
Indexes:
    "scabbard_peer_pkey" PRIMARY KEY, btree (service_id, peer_service_id)
Foreign-key constraints:
    "scabbard_peer_service_id_fkey" FOREIGN KEY (service_id) REFERENCES scabbard_service(service_id)
```

#### SQLite

```sql
CREATE TABLE scabbard_peer (
    service_id       TEXT NOT NULL,
    peer_service_id  TEXT,
    PRIMARY KEY(service_id, peer_service_id),
    FOREIGN KEY(service_id) REFERENCES scabbard_service(service_id)
);
```

### scabbard_service

#### Diesel

```rust
table! {
    scabbard_service (service_id) {
        service_id  -> Text,
        consensus -> Text,
        status -> Text,
    }
}
```

#### PostgreSQL

```
                               Table "public.scabbard_service"
   Column   |             Type             | Collation | Nullable |          Default
------------+------------------------------+-----------+----------+---------------------------
 service_id | text                         |           | not null |
 status     | scabbard_service_status_type |           | not null |
 consensus  | scabbard_consensus           |           | not null | '2PC'::scabbard_consensus
Indexes:
    "scabbard_service_pkey" PRIMARY KEY, btree (service_id)
```

#### SQLite

```sql
CREATE TABLE scabbard_service (
    service_id       TEXT PRIMARY KEY NOT NULL,
    status           TEXT NOT NULL
    CHECK ( status IN ('PREPARED', 'FINALIZED', 'RETIRED') )
, consensus Text NOT NULL DEFAULT '2PC'
  CHECK ( consensus IN ('2PC') ));
```

### scabbard_v3_commit_history

#### Diesel Schema

```rust
table! {
    scabbard_v3_commit_history (service_id, epoch) {
        service_id  -> Text,
        epoch -> BigInt,
        value -> VarChar,
        decision -> Nullable<Text>,
    }
}
```

## Scabbard v3: 2PC Consensus

### Context

A consensus context contains current information maintained by the consensus
algorithm in use (for example, 2PC).

These tables persist specific Augrim data structures.

<div class="mermaid">
erDiagram
    consensus_2pc_context {
        Text service_id PK
        Text coordinator
        BigInt epoch
        BigInt last_commit_epoch
        Text state
        BigInt vote_timer_start
        Text vote
        BigInt decision_timeout_start
    }
    consensus_2pc_context_participant {
        Text service_id PK
        Text process PK
        BigInt epoch
        Text vote
    }
    consensus_2pc_context_participant }o--|| consensus_2pc_context: contains
);
</div>

### Actions

A consensus action is the result of processing an event with a consensus
algorithm, and represents work which must be performed.

<div class="mermaid">
erDiagram
    consensus_2pc_context {}

    consensus_2pc_action {
        INTEGER id PK
        TEXT service_id FK
        TIMESTAMP created_at
        BIGINT executed_at
        INTEGER position
    }
    consensus_2pc_action }o--|| consensus_2pc_context: contains

    consensus_2pc_update_context_action {
        Int8 action_id PK
        Text service_id FK
        Text coordinator
        BigInt epoch
        BigInt last_commit_epoch
        Text state
        BigInt vote_timeout_start
        Text vote
        BigInt decision_timeout_start
        BigInt action_alarm
    }
    consensus_2pc_action ||--o| consensus_2pc_update_context_action: is
    consensus_2pc_update_context_action }o--|| consensus_2pc_context: contains

    consensus_2pc_send_message_action {
        Int8 action_id PK
        Text service_id FK
        BigInt epoch
        Text receiver_service_id
        Text message_type
        Text vote_response
        Binary vote_request
    }

    consensus_2pc_action ||--o| consensus_2pc_send_message_action: is
    consensus_2pc_send_message_action }o--|| consensus_2pc_context: contains

    consensus_2pc_notification_action {
        Int8 action_id PK
        Text service_id FK
        Text notification_type
        Text dropped_message
        Binary request_for_vote_value
    }

    consensus_2pc_action ||--o| consensus_2pc_notification_action: is
    consensus_2pc_notification_action }o--|| consensus_2pc_context: contains

    consensus_2pc_update_context_action_participant {
        Int8 action_id PK
        Text service_id FK
        Text process
        Text vote
    }

consensus_2pc_action ||--o| consensus_2pc_update_context_action_participant: is
consensus_2pc_update_context_action_participant }o--|| consensus_2pc_context: contains
</div>

### Events

A consensus event is input into the consensus algorithm.

<div class="mermaid">
erDiagram
    consensus_2pc_event {
         Int8 id PK
         Text service_id FK
         Timestamp created_at
         BigInt executed_at
         Integer position
         Text event_type
    }

    consensus_2pc_deliver_event {
        Int8 event_id PK
        Text service_id FK
        BigInt epoch
        Text receiver_service_id
        Text message_type
        Text vote_response
        Binary vote_request
    }

    consensus_2pc_event ||--o| consensus_2pc_deliver_event: is
    consensus_2pc_deliver_event }o--|| consensus_2pc_context: contains

    consensus_2pc_start_event {
        Int8 event_id PK
        Text service_id FK
        Binary value
    }

    consensus_2pc_event ||--o| consensus_2pc_start_event: is
    consensus_2pc_start_event }o--|| consensus_2pc_context: contains

    consensus_2pc_vote_event {
        Int8 event_id PK
        Text service_id FK
        Text vote
    }

    consensus_2pc_event ||--o| consensus_2pc_vote_event: is
    consensus_2pc_vote_event }o--|| consensus_2pc_context: contains
</div>

### consensus_2pc_action

#### Diesel

```rust
table! {
    consensus_2pc_action (id) {
        id -> Int8,
        service_id -> Text,
        created_at -> Timestamp,
        executed_at -> Nullable<BigInt>,
        position -> Integer,
    }
}
```

#### PostgreSQL

```
                                         Table "public.consensus_2pc_action"
   Column    |            Type             | Collation | Nullable |                     Default
-------------+-----------------------------+-----------+----------+--------------------------------------------------
 id          | bigint                      |           | not null | nextval('consensus_2pc_action_id_seq'::regclass)
 service_id  | text                        |           | not null |
 created_at  | timestamp without time zone |           | not null | CURRENT_TIMESTAMP
 executed_at | bigint                      |           |          |
 position    | integer                     |           | not null |
Indexes:
    "consensus_2pc_action_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "consensus_2pc_action_service_id_fkey" FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_action (
    id                        INTEGER PRIMARY KEY AUTOINCREMENT,
    service_id                TEXT NOT NULL,
    created_at                TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
    executed_at               BIGINT,
    position                  INTEGER NOT NULL,
    FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
);
```

### consensus_2pc_context

#### Diesel

```rust
table! {
    consensus_2pc_context (service_id) {
        service_id -> Text,
        coordinator -> Text,
        epoch -> BigInt,
        last_commit_epoch -> Nullable<BigInt>,
        state -> Text,
        vote_timeout_start -> Nullable<BigInt>,
        vote -> Nullable<Text>,
        decision_timeout_start -> Nullable<BigInt>,
    }
}
```

#### PostgreSQL

```
                  Table "public.consensus_2pc_context"
         Column         |     Type      | Collation | Nullable | Default
------------------------+---------------+-----------+----------+---------
 service_id             | text          |           | not null |
 coordinator            | text          |           | not null |
 epoch                  | bigint        |           | not null |
 last_commit_epoch      | bigint        |           |          |
 state                  | context_state |           | not null |
 vote_timeout_start     | bigint        |           |          |
 vote                   | text          |           |          |
 decision_timeout_start | bigint        |           |          |
Indexes:
    "consensus_2pc_context_pkey" PRIMARY KEY, btree (service_id)
Check constraints:
    "consensus_2pc_context_check" CHECK (vote_timeout_start IS NOT NULL OR state <> 'VOTING'::context_state)
    "consensus_2pc_context_check1" CHECK ((vote = ANY (ARRAY['TRUE'::text, 'FALSE'::text])) OR state <> 'VOTED'::context_state)
    "consensus_2pc_context_check2" CHECK (decision_timeout_start IS NOT NULL OR state <> 'VOTED'::context_state)
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_context (
    service_id                TEXT PRIMARY KEY,
    coordinator               TEXT NOT NULL,
    epoch                     BIGINT NOT NULL,
    last_commit_epoch         BIGINT,
    state                     TEXT NOT NULL
    CHECK ( state IN ( 'WAITINGFORSTART', 'VOTING', 'WAITINGFORVOTE', 'ABORT', 'COMMIT', 'WAITINGFORVOTEREQUEST', 'VOTED') ),
    vote_timeout_start        BIGINT
    CHECK ( (vote_timeout_start IS NOT NULL) OR ( state != 'VOTING') ),
    vote                      TEXT
    CHECK ( (vote IN ('TRUE' , 'FALSE')) OR ( state != 'VOTED') ),
    decision_timeout_start    BIGINT
    CHECK ( (decision_timeout_start IS NOT NULL) OR ( state != 'VOTED') )
);
```

### consensus_2pc_context_participant

#### Diesel

```rust
table! {
    consensus_2pc_context_participant (service_id, process) {
        service_id -> Text,
        epoch -> BigInt,
        process -> Text,
        vote -> Nullable<Text>,
    }
}
```

#### PostgreSQL

```
   Table "public.consensus_2pc_context_participant"
   Column   |  Type  | Collation | Nullable | Default
------------+--------+-----------+----------+---------
 service_id | text   |           | not null |
 epoch      | bigint |           | not null |
 process    | text   |           | not null |
 vote       | text   |           |          |
Indexes:
    "consensus_2pc_context_participant_pkey" PRIMARY KEY, btree (service_id, process)
Check constraints:
    "consensus_2pc_context_participant_vote_check" CHECK ((vote = ANY (ARRAY['TRUE'::text, 'FALSE'::text])) OR vote IS NULL)
Foreign-key constraints:
    "consensus_2pc_context_participant_service_id_fkey" FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_context_participant (
    service_id                TEXT NOT NULL,
    epoch                     BIGINT NOT NULL,
    process                   TEXT NOT NULL,
    vote                      TEXT
    CHECK ( vote IN ('TRUE' , 'FALSE') OR vote IS NULL ),
    PRIMARY KEY (service_id, process),
    FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
);
```

### consensus_2pc_deliver_event

#### Diesel

```rust
table! {
    consensus_2pc_deliver_event (event_id) {
        event_id -> Int8,
        service_id -> Text,
        epoch -> BigInt,
        receiver_service_id -> Text,
        message_type -> Text,
        vote_response -> Nullable<Text>,
        vote_request -> Nullable<Binary>,
    }
}
```

#### PostgreSQL

```
                    Table "public.consensus_2pc_deliver_event"
       Column        |            Type            | Collation | Nullable | Default
---------------------+----------------------------+-----------+----------+---------
 event_id            | integer                    |           | not null |
 service_id          | text                       |           | not null |
 epoch               | bigint                     |           | not null |
 receiver_service_id | text                       |           | not null |
 message_type        | deliver_event_message_type |           | not null |
 vote_response       | text                       |           |          |
 vote_request        | bytea                      |           |          |
Indexes:
    "consensus_2pc_deliver_event_pkey" PRIMARY KEY, btree (event_id)
Check constraints:
    "consensus_2pc_deliver_event_check" CHECK ((vote_response = ANY (ARRAY['TRUE'::text, 'FALSE'::text])) OR message_type <> 'VOTERESPONSE'::deliver_event_message_type)
    "consensus_2pc_deliver_event_check1" CHECK (vote_request IS NOT NULL OR message_type <> 'VOTEREQUEST'::deliver_event_message_type)
Foreign-key constraints:
    "consensus_2pc_deliver_event_event_id_fkey" FOREIGN KEY (event_id) REFERENCES consensus_2pc_event(id) ON DELETE CASCADE
    "consensus_2pc_deliver_event_service_id_fkey" FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_deliver_event (
    event_id                  INTEGER PRIMARY KEY,
    service_id                TEXT NOT NULL,
    epoch                     BIGINT NOT NULL,
    receiver_service_id       TEXT NOT NULL,
    message_type              TEXT NOT NULL
    CHECK ( message_type IN ('VOTERESPONSE', 'DECISIONREQUEST', 'VOTEREQUEST', 'COMMIT', 'ABORT') ),
    vote_response             TEXT
    CHECK ( (vote_response IN ('TRUE', 'FALSE')) OR (message_type != 'VOTERESPONSE') ),
    vote_request              BINARY
    CHECK ( (vote_request IS NOT NULL) OR (message_type != 'VOTEREQUEST') ),
    FOREIGN KEY (event_id) REFERENCES consensus_2pc_event(id) ON DELETE CASCADE,
    FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
);
```

### consensus_2pc_event

#### Diesel

```rust
table! {
    consensus_2pc_event (id) {
        id -> Int8,
        service_id -> Text,
        created_at -> Timestamp,
        executed_at -> Nullable<BigInt>,
        position -> Integer,
        event_type -> Text,
    }
}
```

#### PostgreSQL

```
                                         Table "public.consensus_2pc_event"
   Column    |            Type             | Collation | Nullable |                     Default
-------------+-----------------------------+-----------+----------+-------------------------------------------------
 id          | bigint                      |           | not null | nextval('consensus_2pc_event_id_seq'::regclass)
 service_id  | text                        |           | not null |
 created_at  | timestamp without time zone |           | not null | CURRENT_TIMESTAMP
 executed_at | bigint                      |           |          |
 position    | integer                     |           | not null |
 event_type  | event_type                  |           | not null |
Indexes:
    "consensus_2pc_event_pkey" PRIMARY KEY, btree (id)
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_event (
    id                        INTEGER PRIMARY KEY AUTOINCREMENT,
    service_id                TEXT NOT NULL,
    created_at                TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
    executed_at               BIGINT,
    position                  INTEGER NOT NULL,
    event_type                TEXT NOT NULL
    CHECK ( event_type IN ('ALARM', 'DELIVER', 'START', 'VOTE') )
);
```

### consensus_2pc_notification_action

#### Diesel

```rust
table! {
    consensus_2pc_notification_action (action_id) {
        action_id -> Int8,
        service_id -> Text,
        notification_type -> Text,
        dropped_message -> Nullable<Text>,
        request_for_vote_value -> Nullable<Binary>,
    }
}
```

#### PostgreSQL

```
              Table "public.consensus_2pc_notification_action"
         Column         |       Type        | Collation | Nullable | Default
------------------------+-------------------+-----------+----------+---------
 action_id              | integer           |           | not null |
 service_id             | text              |           | not null |
 notification_type      | notification_type |           | not null |
 dropped_message        | text              |           |          |
 request_for_vote_value | bytea             |           |          |
Indexes:
    "consensus_2pc_notification_action_pkey" PRIMARY KEY, btree (action_id)
Check constraints:
    "consensus_2pc_notification_action_check" CHECK (dropped_message IS NOT NULL OR notification_type <> 'MESSAGEDROPPED'::notification_type)
    "consensus_2pc_notification_action_check1" CHECK (request_for_vote_value IS NOT NULL OR notification_type <> 'PARTICIPANTREQUESTFORVOTE'::notification_type)
Foreign-key constraints:
    "consensus_2pc_notification_action_action_id_fkey" FOREIGN KEY (action_id) REFERENCES consensus_2pc_action(id) ON DELETE CASCADE
    "consensus_2pc_notification_action_service_id_fkey" FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_notification_action (
    action_id                 INTEGER PRIMARY KEY,
    service_id                TEXT NOT NULL,
    notification_type         TEXT NOT NULL
    CHECK ( notification_type IN ('REQUESTFORSTART', 'COORDINATORREQUESTFORVOTE', 'PARTICIPANTREQUESTFORVOTE', 'COMMIT', 'ABORT', 'MESSAGEDROPPED') ),
    dropped_message           TEXT
    CHECK ( (dropped_message IS NOT NULL) OR (notification_type != 'MESSAGEDROPPED') ),
    request_for_vote_value    BINARY
    CHECK ( (request_for_vote_value IS NOT NULL) OR (notification_type != 'PARTICIPANTREQUESTFORVOTE') ),
    FOREIGN KEY (action_id) REFERENCES consensus_2pc_action(id) ON DELETE CASCADE,
    FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
);
```

### consensus_2pc_send_message_action

#### Diesel

```rust
table! {
    consensus_2pc_send_message_action (action_id) {
        action_id -> Int8,
        service_id -> Text,
        epoch -> BigInt,
        receiver_service_id -> Text,
        message_type -> Text,
        vote_response -> Nullable<Text>,
        vote_request -> Nullable<Binary>,
    }
}
```

#### PostgreSQL

```
          Table "public.consensus_2pc_send_message_action"
       Column        |     Type     | Collation | Nullable | Default
---------------------+--------------+-----------+----------+---------
 action_id           | integer      |           | not null |
 service_id          | text         |           | not null |
 epoch               | bigint       |           | not null |
 receiver_service_id | text         |           | not null |
 message_type        | message_type |           | not null |
 vote_response       | text         |           |          |
 vote_request        | bytea        |           |          |
Indexes:
    "consensus_2pc_send_message_action_pkey" PRIMARY KEY, btree (action_id)
Check constraints:
    "consensus_2pc_send_message_action_check" CHECK ((vote_response = ANY (ARRAY['TRUE'::text, 'FALSE'::text])) OR message_type <> 'VOTERESPONSE'::message_type)
    "consensus_2pc_send_message_action_check1" CHECK (vote_request IS NOT NULL OR message_type <> 'VOTEREQUEST'::message_type)
Foreign-key constraints:
    "consensus_2pc_send_message_action_action_id_fkey" FOREIGN KEY (action_id) REFERENCES consensus_2pc_action(id) ON DELETE CASCADE
    "consensus_2pc_send_message_action_service_id_fkey" FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_send_message_action (
    action_id                 INTEGER PRIMARY KEY,
    service_id                TEXT NOT NULL,
    epoch                     BIGINT NOT NULL,
    receiver_service_id       TEXT NOT NULL,
    message_type              TEXT NOT NULL
    CHECK ( message_type IN ('VOTERESPONSE', 'DECISIONREQUEST', 'VOTEREQUEST', 'COMMIT', 'ABORT') ),
    vote_response             TEXT
    CHECK ( (vote_response IN ('TRUE', 'FALSE')) OR (message_type != 'VOTERESPONSE') ),
    vote_request              BINARY
    CHECK ( (vote_request IS NOT NULL) OR (message_type != 'VOTEREQUEST') ),
    FOREIGN KEY (action_id) REFERENCES consensus_2pc_action(id) ON DELETE CASCADE,
    FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
);
```

### consensus_2pc_start_event

#### Diesel

```rust
table! {
    consensus_2pc_start_event (event_id) {
        event_id -> Int8,
        service_id -> Text,
        value -> Binary,
    }
}
```

#### PostgreSQL

```
       Table "public.consensus_2pc_start_event"
   Column   |  Type   | Collation | Nullable | Default
------------+---------+-----------+----------+---------
 event_id   | integer |           | not null |
 service_id | text    |           | not null |
 value      | bytea   |           |          |
Indexes:
    "consensus_2pc_start_event_pkey" PRIMARY KEY, btree (event_id)
Foreign-key constraints:
    "consensus_2pc_start_event_event_id_fkey" FOREIGN KEY (event_id) REFERENCES consensus_2pc_event(id) ON DELETE CASCADE
    "consensus_2pc_start_event_service_id_fkey" FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_start_event (
    event_id                  INTEGER PRIMARY KEY,
    service_id                TEXT NOT NULL,
    value                     BINARY,
    FOREIGN KEY (event_id) REFERENCES consensus_2pc_event(id) ON DELETE CASCADE,
    FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
);
```

### consensus_2pc_update_context_action

#### Diesel

```rust
table! {
    consensus_2pc_update_context_action (action_id) {
        action_id -> Int8,
        service_id -> Text,
        coordinator -> Text,
        epoch -> BigInt,
        last_commit_epoch -> Nullable<BigInt>,
        state -> Text,
        vote_timeout_start -> Nullable<BigInt>,
        vote -> Nullable<Text>,
        decision_timeout_start -> Nullable<BigInt>,
        action_alarm -> Nullable<BigInt>,
    }
}
```

#### PostgreSQL

```
           Table "public.consensus_2pc_update_context_action"
         Column         |     Type      | Collation | Nullable | Default
------------------------+---------------+-----------+----------+---------
 action_id              | integer       |           | not null |
 service_id             | text          |           | not null |
 coordinator            | text          |           | not null |
 epoch                  | bigint        |           | not null |
 last_commit_epoch      | bigint        |           |          |
 state                  | context_state |           | not null |
 vote_timeout_start     | bigint        |           |          |
 vote                   | text          |           |          |
 decision_timeout_start | bigint        |           |          |
 action_alarm           | bigint        |           |          |
Indexes:
    "consensus_2pc_update_context_action_pkey" PRIMARY KEY, btree (action_id)
Check constraints:
    "consensus_2pc_update_context_action_check" CHECK (vote_timeout_start IS NOT NULL OR state <> 'VOTING'::context_state)
    "consensus_2pc_update_context_action_check1" CHECK ((vote = ANY (ARRAY['TRUE'::text, 'FALSE'::text])) OR state <> 'VOTED'::context_state)
    "consensus_2pc_update_context_action_check2" CHECK (decision_timeout_start IS NOT NULL OR state <> 'VOTED'::context_state)
Foreign-key constraints:
    "consensus_2pc_update_context_action_action_id_fkey" FOREIGN KEY (action_id) REFERENCES consensus_2pc_action(id) ON DELETE CASCADE
    "consensus_2pc_update_context_action_service_id_fkey" FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_update_context_action (
    action_id                 INTEGER PRIMARY KEY,
    service_id                TEXT NOT NULL,
    coordinator               TEXT NOT NULL,
    epoch                     BIGINT NOT NULL,
    last_commit_epoch         BIGINT,
    state                     TEXT NOT NULL
    CHECK ( state IN ( 'WAITINGFORSTART', 'VOTING', 'WAITINGFORVOTE', 'ABORT', 'COMMIT', 'WAITINGFORVOTEREQUEST', 'VOTED') ),
    vote_timeout_start        BIGINT
    CHECK ( (vote_timeout_start IS NOT NULL) OR ( state != 'VOTING') ),
    vote                      TEXT
    CHECK ( (vote IN ('TRUE' , 'FALSE')) OR ( state != 'VOTED') ),
    decision_timeout_start    BIGINT
    CHECK ( (decision_timeout_start IS NOT NULL) OR ( state != 'VOTED') ),
    action_alarm  BIGINT,
    FOREIGN KEY (action_id) REFERENCES consensus_2pc_action(id) ON DELETE CASCADE,
    FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
);
```

### consensus_2pc_update_context_action_participant

#### Diesel

```rust
table! {
    consensus_2pc_update_context_action_participant (action_id) {
        action_id -> Int8,
        service_id -> Text,
        process -> Text,
        vote -> Nullable<Text>,
    }
}
```

#### PostgreSQL

```
Table "public.consensus_2pc_update_context_action_participant"
   Column   |  Type   | Collation | Nullable | Default
------------+---------+-----------+----------+---------
 action_id  | integer |           | not null |
 service_id | text    |           | not null |
 process    | text    |           | not null |
 vote       | text    |           |          |
Indexes:
    "consensus_2pc_update_context_action_participant_pkey" PRIMARY KEY, btree (action_id)
Check constraints:
    "consensus_2pc_update_context_action_participant_vote_check" CHECK ((vote = ANY (ARRAY['TRUE'::text, 'FALSE'::text])) OR vote IS NULL)
Foreign-key constraints:
    "consensus_2pc_update_context_action_participant_action_id_fkey" FOREIGN KEY (action_id) REFERENCES consensus_2pc_action(id) ON DELETE CASCADE
    "consensus_2pc_update_context_action_participant_action_id_fkey1" FOREIGN KEY (action_id) REFERENCES consensus_2pc_update_context_action(action_id) ON DELETE CASCADE
```

#### SQLite

```sql
CREATE TABLE consensus_2pc_update_context_action_participant (
    action_id                 INTEGER PRIMARY KEY,
    service_id                TEXT NOT NULL,
    process                   TEXT NOT NULL,
    vote                      TEXT
    CHECK ( vote IN ('TRUE' , 'FALSE') OR vote IS NULL ),
    FOREIGN KEY (action_id) REFERENCES consensus_2pc_action(id) ON DELETE CASCADE,
    FOREIGN KEY (action_id) REFERENCES consensus_2pc_update_context_action(action_id) ON DELETE CASCADE
);
```

### consensus_2pc_vote_event

### Diesel

```rust
table! {
    consensus_2pc_vote_event (event_id) {
        event_id -> Int8,
        service_id -> Text,
        vote -> Text,
    }
}
```

### PostgreSQL

```
        Table "public.consensus_2pc_vote_event"
   Column   |  Type   | Collation | Nullable | Default
------------+---------+-----------+----------+---------
 event_id   | integer |           | not null |
 service_id | text    |           | not null |
 vote       | text    |           |          |
Indexes:
    "consensus_2pc_vote_event_pkey" PRIMARY KEY, btree (event_id)
Check constraints:
    "consensus_2pc_vote_event_vote_check" CHECK (vote = ANY (ARRAY['TRUE'::text, 'FALSE'::text]))
Foreign-key constraints:
    "consensus_2pc_vote_event_event_id_fkey" FOREIGN KEY (event_id) REFERENCES consensus_2pc_event(id) ON DELETE CASCADE
    "consensus_2pc_vote_event_service_id_fkey" FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
```

### SQLite

```sql
CREATE TABLE consensus_2pc_vote_event (
    event_id                  INTEGER PRIMARY KEY,
    service_id                TEXT NOT NULL,
    vote                      TEXT
    CHECK ( vote IN ('TRUE' , 'FALSE') ),
    FOREIGN KEY (event_id) REFERENCES consensus_2pc_event(id) ON DELETE CASCADE,
    FOREIGN KEY (service_id) REFERENCES consensus_2pc_context(service_id) ON DELETE CASCADE
);
```

### Example Queries

#### Listing all actions

```sql
SELECT id,
       consensus_2pc_action.service_id,
       consensus_2pc_notification_action.notification_type as n_notification_type,
       consensus_2pc_notification_action.dropped_message as n_dropped_message,
       consensus_2pc_notification_action.request_for_vote_value as n_request_for_vote_value,
       consensus_2pc_update_context_action.coordinator as uc_coordinator,
       consensus_2pc_update_context_action.epoch as uc_epoch,
       consensus_2pc_update_context_action.last_commit_epoch as uc_last_commit_epoch,
       consensus_2pc_update_context_action.state as uc_state,
       consensus_2pc_update_context_action.vote_timeout_start as uc_vote_timeout_start,
       consensus_2pc_update_context_action.vote as uc_vote,
       consensus_2pc_update_context_action.decision_timeout_start as uc_decision_timout_start,
       consensus_2pc_update_context_action.action_alarm as uc_action_alarm,
       consensus_2pc_update_context_action_participant.process as ucp_process,
       consensus_2pc_update_context_action_participant.vote as ucp_vote,
       created_at,
       executed_at
FROM consensus_2pc_action
LEFT JOIN consensus_2pc_notification_action ON consensus_2pc_action.id=consensus_2pc_notification_action.action_id
LEFT JOIN consensus_2pc_update_context_action ON consensus_2pc_action.id=consensus_2pc_update_context_action.action_id
LEFT JOIN consensus_2pc_update_context_action_participant ON consensus_2pc_action.id=consensus_2pc_update_context_action_participant.action_id
ORDER BY id;
```