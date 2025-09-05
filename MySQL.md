# 📂 Database Schema

## USERS
```sql
CREATE TABLE users (
  id             BIGINT AUTO_INCREMENT PRIMARY KEY,
  username       VARCHAR(255) NOT NULL UNIQUE,
  password       VARCHAR(255) NOT NULL,
  nickname       VARCHAR(255) NOT NULL,
  birth_date     DATE NOT NULL,
  email          VARCHAR(255) NOT NULL UNIQUE,
  refresh_token  VARCHAR(500) NULL,
  created_at     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```
## inventories
```sql
CREATE TABLE inventories (
  id          BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id     BIGINT NOT NULL UNIQUE,
  created_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_inventories_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE
);
```
## inventory_items
```sql
CREATE TABLE inventory_items (
  inventory_id BIGINT NOT NULL,
  item_code    VARCHAR(100) NOT NULL,
  amount       INT NOT NULL,
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (inventory_id, item_code),
  CONSTRAINT fk_inv_items_inventory
    FOREIGN KEY (inventory_id) REFERENCES inventories(id)
    ON DELETE CASCADE
);
```
## inventory_weapons
```sql
CREATE TABLE inventory_weapons (
  inventory_id BIGINT NOT NULL,
  weapon_code  VARCHAR(100) NOT NULL,
  amount       INT NOT NULL,
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (inventory_id, weapon_code),
  CONSTRAINT fk_inv_weapons_inventory
    FOREIGN KEY (inventory_id) REFERENCES inventories(id)
    ON DELETE CASCADE
);
```
## friend_requests
```sql
CREATE TABLE friend_requests (
  id           BIGINT AUTO_INCREMENT PRIMARY KEY,
  requester_id BIGINT NOT NULL, -- 보낸 사람
  receiver_id  BIGINT NOT NULL, -- 받은 사람
  status       ENUM('PENDING','DECLINED','CANCELLED') NOT NULL DEFAULT 'PENDING'
               COMMENT 'PENDING: 대기, DECLINED: 수신자 거절, CANCELLED: 요청자 취소',
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_fr_req_requester FOREIGN KEY (requester_id) REFERENCES users(id) ON DELETE CASCADE,
  CONSTRAINT fk_fr_req_receiver  FOREIGN KEY (receiver_id) REFERENCES users(id) ON DELETE CASCADE,
  CONSTRAINT uq_friend_request UNIQUE (requester_id, receiver_id),
  INDEX idx_fr_req_reverse (receiver_id, requester_id)
);
```
## friends
```sql
CREATE TABLE friends (
  id         BIGINT AUTO_INCREMENT PRIMARY KEY,
  user1_id   BIGINT NOT NULL,
  user2_id   BIGINT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_friends_user1 FOREIGN KEY (user1_id) REFERENCES users(id) ON DELETE CASCADE,
  CONSTRAINT fk_friends_user2 FOREIGN KEY (user2_id) REFERENCES users(id) ON DELETE CASCADE,
  CONSTRAINT uq_friend_pair UNIQUE (user1_id, user2_id),
  CHECK (user1_id < user2_id)
);
```
## flag_data
```sql
CREATE TABLE flag_data (
  user_id    BIGINT NOT NULL,
  flag_code  VARCHAR(100) NOT NULL,
  flag_state BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (user_id, flag_code),
  CONSTRAINT fk_flag_data_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```