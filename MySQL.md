-- ====== cnwvdb_s0 ======
USE cnwvdb_s0;

-- ====== cnwvdb_s1 ======
USE cnwvdb_s1;

SET FOREIGN_KEY_CHECKS=0;
DROP TABLE IF EXISTS friends;
DROP TABLE IF EXISTS inventory_weapons;
DROP TABLE IF EXISTS inventory_items;
DROP TABLE IF EXISTS inventories;
DROP TABLE IF EXISTS flag_data;
DROP TABLE IF EXISTS users;
DROP TABLE IF EXISTS friend_requests;
SET FOREIGN_KEY_CHECKS=1;

-- USERS
CREATE TABLE users (
  id             BIGINT AUTO_INCREMENT PRIMARY KEY,
  username       VARCHAR(255) NOT NULL UNIQUE,
  password       VARCHAR(255) NOT NULL,
  nickname       VARCHAR(255) NOT NULL,
  birth_date     DATE NOT NULL,
  email          VARCHAR(255) NOT NULL UNIQUE,
  refresh_token  VARCHAR(500) NULL,
  created_at     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- INVENTORIES (1:1)
CREATE TABLE inventories (
  id          BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id     BIGINT NOT NULL UNIQUE,
  created_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_inventories_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- INVENTORY_ITEMS (복합 PK) + version
CREATE TABLE inventory_items (
  inventory_id BIGINT NOT NULL,
  item_code    VARCHAR(100) NOT NULL,
  amount       INT NOT NULL,
  version      BIGINT NOT NULL DEFAULT 0,
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (inventory_id, item_code),
  CONSTRAINT fk_inv_items_inventory
    FOREIGN KEY (inventory_id) REFERENCES inventories(id)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- INVENTORY_WEAPONS (복합 PK) + version
CREATE TABLE inventory_weapons (
  inventory_id BIGINT NOT NULL,
  weapon_code  VARCHAR(100) NOT NULL,
  amount       INT NOT NULL,
  version      BIGINT NOT NULL DEFAULT 0,
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (inventory_id, weapon_code),
  CONSTRAINT fk_inv_weapons_inventory
    FOREIGN KEY (inventory_id) REFERENCES inventories(id)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- FLAG_DATA (복합 PK, 예약어 피해서 flag_state)
CREATE TABLE flag_data (
  user_id    BIGINT NOT NULL,
  flag_code  VARCHAR(100) NOT NULL,
  flag_state BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (user_id, flag_code),
  CONSTRAINT fk_flag_data_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- FRIENDS (엔티티 Friend와 일치: user_id / friend_id / status VARCHAR)
CREATE TABLE friends (
  id         BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id    BIGINT NOT NULL,
  friend_id  BIGINT NOT NULL,
  status     VARCHAR(50) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_friends_user   FOREIGN KEY (user_id)   REFERENCES users(id) ON DELETE CASCADE,
  CONSTRAINT fk_friends_friend FOREIGN KEY (friend_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


-- FRIEND REQUESTS
CREATE TABLE friend_requests (
  id           BIGINT AUTO_INCREMENT PRIMARY KEY,
  requester_id BIGINT NOT NULL, -- 보낸 사람
  receiver_id  BIGINT NOT NULL, -- 받은 사람
  status       ENUM('PENDING','DECLINED','CANCELLED') NOT NULL DEFAULT 'PENDING'
               COMMENT 'PENDING: 대기, DECLINED: 수신자 거절, CANCELLED: 요청자 취소',
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_fr_req_requester FOREIGN KEY (requester_id) REFERENCES users(id) ON DELETE CASCADE,
  CONSTRAINT fk_fr_req_receiver  FOREIGN KEY (receiver_id)  REFERENCES users(id) ON DELETE CASCADE,
  CONSTRAINT uq_friend_request UNIQUE (requester_id, receiver_id),
  INDEX idx_fr_req_reverse (receiver_id, requester_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS player_stats (
  user_id       BIGINT      NOT NULL,
  hp            INT         NOT NULL DEFAULT 100,
  atk           INT         NOT NULL DEFAULT 10,
  spd           INT         NOT NULL DEFAULT 10,
  hit_rate      INT         NOT NULL DEFAULT 70,
  evasion_rate  INT         NOT NULL DEFAULT 12,
  counter_rate  INT         NOT NULL DEFAULT 3,
  updated_at    TIMESTAMP   NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  version       BIGINT      NOT NULL DEFAULT 0,
  PRIMARY KEY (user_id),
  CONSTRAINT fk_player_stats_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE
);