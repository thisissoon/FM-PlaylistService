# SOON\_ FM Playlist Service

Management of the SOON_ FM Playlist, allowing for add, remove, and a score mechanic.

## Playlist Management

All REST endpoints will follow the HAL JSON specification.

### GET /

This will return the current playlist, paginated. The current playing track is **NOT** included, see `GET /current`

``` json
{
  "_embedded": [
    {
      "id": "sfm:playlist:123456789",
      "spotify_uri": "spotify:track:123456789",
    },  
    {
      "id": "sfm:playlist:987654321",
      "spotify_uri": "spotify:track:987654321",
    },
  ],
  "_links": {
    "self": {
      "title": "Page 1 of Playlist Tracks",
      "href": "/",
    },
    "next": {
      "title": "Page 2 of Playlist tracks",
      "href": "/?page=2",
    }
  },
  "page": 1,
  "total": 20,
  "limit": 10,
}
```

### GET /current

Returns the current playing track as set by the player via a push event.

``` json
{
  "_links": {
    "self": {
      "title": "Current Track",
      "href": "/current",
    }
  },
  "uri": "spotify:track:123456789",
}
```

### POST /

Add a new track to the end of the playlist.

#### Request

``` http
POST /
Content-Type: application/json

{
  "spotify_uri": "spotify:track:123456789",
  "user_id": "123456789",
}
```

#### Response

This resource will respond with an empty body. The `Location` header will return a URI to the created object.

``` http
201 Created
Location /123456789
```

### PUT /:playlist_id

Update a playlist item. this allows changes of the following:

* Set a tracks skipped state (skipped or not skipped)
* Set a tracks played state (played or not played)
* Set a tracks queued state (queued or not queued)

``` json
{
  "played": true,
  "skipped": true,
  "queued": false,
}
```

Storing specific track states in this way will allow us to for example find trackes played but then skipped or tracked queued but voted out.

### DELETE /:playlist_id

Remove an item from the playlist. Returns an empty `410 Gone`. This will usually be from the playlist item creator.

``` http
410 Gone
```

## Scoring Mechanic

Each item in the playlist can be up or down votes. Once a playlist item reaches a threshold if negative votes it will be removed from the playlist. Users will now be ranked based on a combination of the volumn of track plays but also the quality. The score system is as such:

- Add a track: +1 Point
- Up vote on a playlist item: +3
- Down vote on a playlist item: -3

### POST /:playlist_id/vote

Add an upvote to the playlist item.

Returns an empty `201 Created`

### DELETE /:playlist_id/vote

Add a downvote to a playlist item.

Returns an empty `201 Created` or `410 Gone` in the case a track is removed from the queue.
