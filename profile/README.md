# Monopoly 3D

# Packet Classification

### BasePacket

Base packet represents universal data and metadata transmission format for both client and server packets.

```json
{
  "data": {},
  "meta": {
    "tag": "packet_tag",
    "class": "packet_class"
  }
}
```

Packet meta is a list of values which both client and server use to authorize and verify incoming packets, and to provide different implementations for handling them.

Packet tag is a string tag of client name and packet class is a classification of sender, whether it is a client or server.

### ClientPacket

Client packet represents a format of all packets which are sent by client.

```json
{
  "data": {},
  "meta": {
    "tag": "packet_tag",
    "class": "client"
  }
}
```

All client packets must have ```client``` packet class.

### ServerPacket

Server packet represents a format of all packets which are sent by server.

```json
{
  "data": {},
  "meta": {
    "tag": "packet_tag",
    "class": "server"
  }
}
```

All server packets must have ```server``` packet class.

## Client Packets

### ClientAuthPacket

```json
{
  "data": {
    "ticket": "JWT-encoded ticket"
  },
  "meta": {
    "tag": "auth",
    "class": "client"
  }
}
```

Authenticates client, must be the first packet sent to server, otherwise server will close the connection.

```Ticket``` is a JWT-encoded token that authenticates client by user ID, client must get this ticket by sending a specific HTTP request.

### ClientPingPacket

```json
{
  "data": {
    "request": "Message"
  },
  "meta": {
    "tag": "ping",
    "class": "client"
  }
}
```

Sends a simple text message to server in request field, should be sent every 5 seconds to maintain the connection.

### ClientPlayerJoinGamePacket

```json
{
  "data": {
    "game_id": "UUID"
  },
  "meta": {
    "tag": "player_join_game",
    "class": "client"
  }
}
```

Sent when a player joins the game.

### ClientPlayerMovePacket

```json
{
  "data": {
    "game_id": "UUID"
  },
  "meta": {
    "tag": "player_move",
    "class": "client"
  }
}
```

Sent when a player rolls the dices and moves.

### ClientPlayerReadyPacket

```json
{
  "data": {
    "game_id": "UUID",
    "is_ready": "bool"
  },
  "meta": {
    "tag": "player_ready",
    "class": "client"
  }
}
```

Sent when a player chooses whether they are ready to play or not.

## Server Packets

### ServerAuthResponsePacket

```json
{
  "data": {
    "user_id": "UUID",
    "username": "Plummy"
  },
  "meta": {
    "tag": "auth",
    "class": "server"
  }
}
```

Sends a response if the authentication succeeded, otherwise the server closes the connection with code 3000.

### ServerErrorPacket

```json
{
  "data": {
    "status": 4000,
    "detail": "Error detail"
  },
  "meta": {
    "tag": "server_error",
    "class": "server"
  }
}
```

Sends an exception status code and detail.

### ServerGameCountdownStartPacket

```json
{
  "data": {
    "game_id": "UUID",
    "delay": 10
  },
  "meta": {
    "tag": "game_countdown_start",
    "class": "server"
  }
}
```

Sent when the game start countdown starts. ```Delay``` is an integer that represents the delay in seconds before the game starts

### ServerGameCountdownStopPacket

```json
{
  "data": {
    "game_id": "UUID"
  },
  "meta": {
    "tag": "game_countdown_stop",
    "class": "server"
  }
}
```

Sent when the game start countdown stops.

### ServerGameMovePacket

```json
{
  "data": {
    "game_id": "UUID",
    "round": 0,
    "move": 0
  },
  "meta": {
    "tag": "game_move",
    "class": "server"
  }
}
```

Sent when the previous player has finished their turn and the game is now waiting for the next player to move. ```Round``` is the current round number, and ```move``` is the index of the player the game is waiting for to move.

### ServerGameStartPacket

```json
{
  "data": {
    "game_id": "UUID",
    "players": [
      {
        "player_id": "UUID",
        "balance": 15000,
        "field": 0
      }
    ],
    "fields": [
      {
        "field_id": 0,
        "field_type": "company | chance | start | tax | prison | police | casino"
      }
    ]
  },
  "meta": {
    "tag": "game_start",
    "class": "server"
  }
}
```

Sent when the game starts.

```Players``` is a list of all players, where ```balance``` is every player's total bank balance, and ```field``` is a field index where each player is standing on.

```Fields``` is a list of all fields, where ```field_id``` is an index of each field on the map.

Depending on the field type each field may have additional appropriately named attribute.

```json
{
  "field_id": 0,
  "field_type": "company",
  "company": {
    "owner_id": "UUID or null",
    "is_monopoly": "bool",
    "field_dependant": "bool",
    "dice_dependant": "bool",
    "rent": 1000,
    "mortgage": -1,
    "filiation": 0,
    "cost": 5000,
    "mortgage_cost": 2500,
    "buyout_cost": 3000,
    "filiation_cost": 1000
  }
}
```

```json
{
  "field_id": 0,
  "field_type": "tax",
  "tax": {
    "tax_amount": 1000
  }
}
```

### ServerPingPacket

```json
{
  "data": {
    "response": "Message"
  },
  "meta": {
    "tag": "server_ping",
    "class": "server"
  }
}
```

Sends a simple text message to client in response field.

### ServerPlayerBuyFieldPacket

```json
{
  "data": {
    "game_id": "UUID",
    "player_id": "UUID",
    "field": 0,
    "cost": 1000
  },
  "meta": {
    "tag": "player_buy_field",
    "class": "server"
  }
}
```

Sent when a player is awaited to buy a field or to put it up for auction. ```Field``` is an index of a field and ```cost``` is a price for which a player can buy it.

### ServerPlayerGotImprisonedPacket

```json
{
  "data": {
    "game_id": "UUID",
    "player_id": "UUID",
    "field": 0,
    "imprison_cause": "police | chance | double"
  },
  "meta": {
    "tag": "player_got_imprisoned",
    "class": "server"
  }
}
```

Sent when a player was imprisoned. ```Field``` is an index of a field the player was moved to (An index of a prison field) and ```imprison cause``` is an imprison variant, whether it was because player stepped on a police field, got an unlucky chance or made 3 doubles in a row.

### ServerPlayerGotStartBonusPacket

```json
{
  "data": {
    "game_id": "UUID",
    "player_id": "UUID",
    "balance": 15000
  },
  "meta": {
    "tag": "player_got_start_bonus",
    "class": "server"
  }
}
```

Sent when a player has got a bonus for passing the start field. ```Balance``` is a player's resulted bank balance.

### ServerPlayerGotStartRewardPacket

```json
{
  "data": {
    "game_id": "UUID",
    "player_id": "UUID",
    "balance": 15000
  },
  "meta": {
    "tag": "player_got_start_reward",
    "class": "server"
  }
}
```

Sent when a player has got a reward for moving directly onto a start field. ```Balance``` is a player's resulted bank balance.

### ServerPlayerGotTaxPacket

```json
{
  "data": {
    "game_id": "UUID",
    "player_id": "UUID",
    "amount": 1000
  },
  "meta": {
    "tag": "player_got_tax",
    "class": "server"
  }
}
```

Sent when a player moves onto a tax field. ```Amount``` is a tax amount the player must pay.

### ServerPlayerJoinGamePacket

```json
{
  "data": {
    "game_id": "UUID",
    "player": {
      "id": "UUID",
      "username": "Deyman Qr"
    }
  },
  "meta": {
    "tag": "player_join_game",
    "class": "server"
  }
}
```

Sent when a player joins the game.

### ServerPlayerMovePacket

```json
{
  "data": {
    "game_id": "UUID",
    "player_id": "UUID",
    "dices": [1, 2],
    "field": 0
  },
  "meta": {
    "tag": "player_move",
    "class": "server"
  }
}
```

Sent when player rolls the dices and makes a move. ```Dices``` is a two-sized list with a throw result on each dice and ```field``` is a field index a player has moved onto.

### ServerPlayerPayRentPacket

```json
{
  "data": {
    "game_id": "UUID",
    "player_id": "UUID",
    "field": 0,
    "amount": 1000
  },
  "meta": {
    "tag": "player_pay_rent",
    "class": "server"
  }
}
```

Sent when player must pay rent for visiting other player's field. ```Field``` is a field index a player must pay rent for and ```amount``` is an amount that player must pay.

### ServerPlayerReadyPacket

```json
{
  "data": {
    "game_id": "UUID",
    "player_id": "UUID",
    "is_ready": "bool"
  },
  "meta": {
    "tag": "player_ready",
    "class": "server"
  }
}
```

Sent when player decides whether they are ready to play or not. 
