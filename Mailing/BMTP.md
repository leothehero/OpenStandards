# Basic Mail Transfer Protocol
This **draft** is the reference of BMTP 1.0, BMTP is an e-mail protocol that allows embeds, accounts and mail directories.
This document however doesn't describes how server should send mail data to other servers. This process is described by Basic Inter-Server Mail Transfer Protocol (BISMTP).
This means that BMTP can works with servers that have another way of transfering mails between servers, and it also means that BISMTP can be used alongside any other protocol for distributing mail to
the clients.

## Definitions
- E-Mail Address: The standard e-mail address you use everyday (e.g.: zen1th@gertiscool.com)
- Mail: Text with: destination, source, subject and optional embedded files
- Command: A command is an action sent from client to server, it is wrote as text equivalents on the document but must be replaced by their binary equivalents in actual sending.

## Protocol
### Response format
Responses to client commands are sent in a way to allow parallel operations on the same socket.
The decoded response is always in the form of a Lua table, no matter the data, if part or all the data is in a binary format (outside valid string characters), they still must be stored in the string no matter the way, however, before the binary serialized table is sent, the byte corresponding to the command it responds is sent.
The Lua table contains two entries the first contains the result of the command, while the second is the exact command it is responding to (in their corresponding byte, not name!) effectively allowing for a client to send many commands at once, and get the response of the corresponding commands. The Server doesn't need to be async and can return the commands synchronously, which to the client will just appear as the server responding to the commands in the same order as they were sent, servers can also mix both to limit the number of threads, or always do async, they are free of choice.

### Serialization
While "Response format" section defines how a response is internally defined, it doesn't define how it is sent on the network:
All numbers larger than 8 bytes are in **big endian**.

Entries:
- Start array: `0x01` (followed by more entries)
- End array: `0x02` (the end of the last array, note that arrays are recursive)
- String: `0x03` followed by the **BYTE** length in `uint32` (unsigned 32-bit integer), and then the data
- String (length smaller than `0xFFFF`): `0x04`: followed by the byte length in `uint16` followed by the data
- String (length smaller than `0xFF`): `0x05`: followed by the byte length in byte and then the data
- Number (integer): `0x06` followed by `uint32` of the number
- Byte (any integer lower than `0xFF`): `0x07` followed by the unsigned byte

### Commands

#### `LOGIN`
The byte corresponding to `LOGIN` is `0x04`.

Logging in is necessary in all sockets before any other MAIL command.  
The password is sent as a **binary** SHA3-256 hash. The server should not store this hash as the password but
the hash of the hash (why? because why not).

Command:
  Arguments: `email password_hash`

Example:
```
--> {0x04}
```


#### `SEND`
The byte corresponding to `SEND` is `0x01`.

Send:
The server knows the sender field of the email by the previous `LOGIN`

arguments: `destination subject content`

```
--> {0x01, "test@test.com", "Hey!", "i'm blablabla\nblablabla. wowow blabla"}
```

#### `ESEND`
The byte corresponding to `ESEND` is `0x02`.

Mails that have embedded files are sent via the `ESEND` (Embed Send) command, which is very similar to `SEND`:
arguments: `destination subject content embeds`
`embeds` is an array of embed.
An embed is an array of 2 strings: the first is the file name and the second is the data.

```
--> {0x03, "test@test.com", "Hey!", "blablabla", { {"test.arc", "binary data"}, {"test.txt", "some text document"} } }
```

#### `LIST`
The byte corresponding to `LIST` is `0x03`.

Listing is done by sending the `LIST` command.
`LIST` command takes an optional argument: `directory`, when ommited `LIST` will return the list of directories.  
When `directory` argument is included, it will return the metadatas about Mail in the specified `directory`.
A listed mail has in the following order: subject, source, reference ID.

The return format is the same

Example (unserialized) (--> = client request; <-- = server response):
```
--> {0x03}
<-- {
		{0x03},
		{"Trash", "Recent", "All"}
	}
--> {0x03, "Trash"}
<-- {
		{0x03, "Trash"},
		{
			"Hey friend!",
			"tlb@endermail.com",
			98
		}
	}
```

## `DOWN`
The byte corresponding to `DOWN` is `0x05`.

This command allow to download the content, subject, source and embeds if present of an email.
The email is located with its server reference ID.

Example (unserialized):
```
--> {0x05, 98}
<-- {
		{0x05},
		{
			"Hey friend!",
			"tlb@endermail.com",
			"Hey my friend i just tested the BMTP protocol through its official implementation on GERT!\nIt's so much amazing! I integrated you a DFPWM audio.",
			{
				{"noises.dfpwm", "\x98\x22\xFF\x7F\x3F\x4D ..."}
			}
		}
	}
```
