# DocumentService - strictly typed datastores!

DocumentService is a luau library for saving data with Roblox datastores. DocumentStores hold data about an entity.
This project is currently in early stages and should not be used.

DocumentService does not use promises. This enables better type support and allows you to choose which coroutine abstraction you use, if any.

## Features
- Full strict typing
- Data validation and migration
- Simple session locking API
- Run hooks before and after operations, e.g. logging
- Automatic retry with exponential backoff
- Automatically uses existing data
- Checks your data can be stored in JSON

## Long-term goals
- ACID transactions involving multiple Documents
- Local data with auto-save for session locked data (e.g. ReadLocal and WriteLocal methods)
- Queue and throttle transactions, yielding the thread while queued

## Inspiration
This library takes inspiration from Lapis, ProfileService, keyForm and others. These are all great projects but didn't meet my needs in some way.

## Contributing
Contributions are accepted. Your contributions must run in Lune and, when compiled by darklua to target/roblox, in Roblox.

## Contact
- The best way to get in touch is to ping me in the thread in the Roblox OSS discord, or create an issue.
