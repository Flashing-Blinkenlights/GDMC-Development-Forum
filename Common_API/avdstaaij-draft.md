Initial thoughts for the common HTTP API
========================================

## Must have:

- **Place a block**\
  `PUT /blocks/<x,y,z>`\
  It should be possible to pass block states and NBT data in the request body.
  Passing NBT data is currently not supported.

- **Place blocks asynchronously**\
  `POST /blocks`\
  This should have feature parity with the single-block version.
  I think `POST` fits better than `PUT` here since we're not directly updating a
  block resource, but rather asking the server to do it in the background.
  We are essentially submitting a task.
  One could even argue that this should be `POST /block-tasks` and that it
  should return a task identifier that can be used to get the progress
  (`GET /block-tasks/<identifier>`) or to abort the task
  (`DELETE /block-tasks/<identifier>`).

- **Synchronise with block placement tasks**\
  I'm not sure what the most REST way to do this would be. Some options:
  - `GET /sync`, which blocks until all placements are complete.
  - `GET /block-tasks`, which does not block, but returns whether there are
    still outstanding tasks. Clients could then use this in a loop. This
    combines well with the `/block-tasks/<identifier>` API described above.

- **Get a block**\
  `GET /blocks/<x,y,z>`

- **Get blocks**\
  `GET /blocks/<x1,x2,y1,y2,z1,z2>` or
  `GET /blocks/<x1,y1,z1,x2,y2,z2>`

- **Get the buildarea**\
  `GET /buildarea`\
  I suggest to enforce the buildarea in the backend. I expect the cost to be
  negligible, especially when clients use batched placement.
  Another idea is to interpret all block requests as relative to the buildarea
  automatically.


## Should have:

- **Get biome(s)**\
  Same API as getting blocks, but on `GET /biomes`.

- **Get heightmap**\
  `GET /chunk/<chunkX,chunkY,chunkZ>/heightmaps/<heightmap>`

- **Execute command**\
  `POST /command`\
  This is a must-have if setting NBT data with the block-setting endpoint is
  not supported.

- **Get Minecraft version**\
  `GET /version`\
  GDPC currently parses the version from requested chunks. While it is nice that
  this is possible, it is very low-level, and it might break in future versions.
  It's probably trivial to implement this in the backend, so why not make an
  endpoint for it?

- **Get chunk**\
  `GET /chunk/<chunkX,chunkY,chunkZ>/data` (or `/raw`, `/nbt`, etc.)\
  A low-level request to get a chunk in NBT format. We should probably provide
  higher-level requests like `GET /biomes`, but this low-level request might
  still be useful. Make the easy things easy, and the hard things possible.
  This is a must-have if the biome-getting endpoint is not supported.

## Could have:

- **Get other chunk data**\
  `GET /chunk/<chunkX,chunkY,chunkZ>/...`\
  For other chunk data like heightmaps:
  [https://minecraft.fandom.com/wiki/Chunk_format](https://minecraft.fandom.com/wiki/Chunk_format)
  These are higher-level alternatives to getting the full NBT data.

- **Set buildarea**\
  `PUT /buildarea`\
  Setting the build area from code might be useful. To test more rapidly, for
  example. We might want to add a way to block this request if we want to
  automate running GDMC submissions.

- **Read/manipulate static entities like item frames and armor stands**\
  I have no suggestion for the API here.

- **Read/manipulate mobs or players**\
  I have no suggestion for the API here.
