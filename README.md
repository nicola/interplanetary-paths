# merkle-paths

WIP Hierarchical pathing scheme to traverse in and across merkle trees - merkle dags. (eventually IPLD)

## Note

After a great talk with some students at MIT, I realized that this scheme is not just valid for Merkle trees, but for any type of structures. It is a way to link across data (like the Linked Data model). If we edit `MerkleLink` to `HTTPLink`, then we can link data via the web, instead.

## Pathing examples

### Without hash/merkle links

```javascript
// _hash_
{
  "name": "Nicola",
  "friends": [{
    name: "Adam"
  }]
}


// /_hash_/name
"Nicola"

// /_hash_/friends
[{
  name: "Adam"
}]

// /_hash_/friends/0
{
  name: "Adam"
}

// /_hash_/friends/0/name
"Adam"

```

### With hash links

```javascript
// _hash1_
{
  "name": "Nicola",
  "surname": MerkleLink({@link: _hash3_})
  "friends": [MerkleLink({
    @link: _hash2_
  })]
}

// _hash2_
{
  name: "Adam"
}

// _hash3_
"Greco"

// /_hash1_/surname
"Greco"

// /_hash1_/friends
[MerkleLink({
  @link: _hash2_
})]

// /_hash1_/friends/0
{
  name: "Adam"
}

// /_hash1_/friends/0/name
"Adam"
```

### With data about the link

```javascript
// _hash1_
{
  "name": "Nicola",
  "friends": [MerkleLink({
    @link: _hash_2,
    nickname: "yala"
  })]
}

// _hash2_
{
  name: "Adam"
}

// /_hash1_/friends
[MerkleLink({
  @link: _hash_2,
  nickname: "yala"
})]

// /_hash1_/friends/0
{
  name: "Adam"
}

// /_hash1_/friends/0/name
"Adam"

// /_hash1_/friends/0/nickname
undefined

// /_hash1_/friends/0#nickname
"yala"
```

### With merkle links

```javascript
// _hash1_
{
  name: /_hash3_/name
  friends: [
    MerkleLink({
      @link: /_hash2_/name
    })
  ]
}

// _hash2_
{
  name: {
    first: /_hash3_/name,
    family: "Greco"
  }
}

// _hash3_
{
  name: "Nicola"
}

// /_hash1_/name
"Nicola"

// /_hash1_/friends/0
{
  first: "Nicola",
  family: "Greco"
}

// /_hash1_/friends/0/first
"Nicola"

```

## Relative graphs (cycle)

Cyclic graphs can be created using relative paths

```javascript
// _hash1_
{
  name: "Nicola",
  surname: MerkleLink(@link: "./passport/officialSurname")
  passport: {
    officialName: MerkleLink(@link: "../name"),
    officialSurname: "Greco"
  }
}

// /_hash1_/name
"Nicola"

// /_hash1_/surname
"Greco"

// /_hash1_/passport
{
  officialName: MerkleLink(@link: "../name"),
  officialSurname: "Greco"
}

// /_hash1_/passport/officialName
"Nicola"
```

### Cycles in merkle graphs

```javascript
// _hash1_
{
  nicola: {
    name: "Nicola"
    sister: MerkleLink({@link: "../nicola"})
  },
  lucia: {
    name: "Lucia"
    brother: MerkleLink({@link: "../lucia"})
  }
}

// _hash2_
{
  nicola: _hash3_,
  lucia: _hash4_
}

// _hash3_
{
  name: "Lucia"
  brother: MerkleLink({@link: "../lucia"})
}

// _hash4_
{
  name: "Nicola"
  sister: MerkleLink({@link: "../nicola"})
}

// /_hash1_/nicola/sister/name
{
  name: "Lucia"
}

// /_hash1_/nicola/sister/brother/name
{
  name: "Nicola"
}

// /_hash2_/nicola/sister/name
{
  name: "Lucia"
}

// /_hash2_/nicola/sister/brother/name
{
  name: "Nicola"
}
```

## Mutable paths!

## Implementation note

`MerkleLink` in the example describe the fact that that branch of the object should be treated differently

Example implementations:
- In CBOR, it can just be a tag
- In JavaScript, one could just check if the property is an object and contains `@link` inside


### Efficient encoding

It may be a good idea to store the data in the following way: list the links before the data.
In this way navigating through data will be cheap (no need to retrieve the _ENTIRE_ content, one should just do a binary search in the links (if the number of the links is stored at the top of the entry), otherwise linear)

```
{
  name: hash2,
  surname: "Greco"
  friends: [{
    name: MerkleLink(hash1/name),
    surname: "Yala
  }]
}
+------------------------------+
| ./friends/0/name: hash1/name |
| ./name: hash2                |
+------------------------------+
| surname: "Nicola"            |
| friends/0/surname: "Yala"    |
+------------------------------+

// or, in other words

+------------------------------+
| {                            |
|   friends: [{                |
|     name: hash1/name         |
|   }],                        |
|   name: hash2                |
| }                            |
+------------------------------+
| {                            |
|  surname: "Greco"            |
|  friends: [{                 |
|    surname: "Yala"           |
|  }]                          |
| }                            |
+------------------------------+
