While gun's conflict resolution algorithm does not require clocks to be in sync, synced clocks are still very useful in distributed systems. In gun, you can use the [Network Time Sync (NTS) library](https://github.com/amark/gun/blob/master/nts.js) built into gun to sync clocks across peers. This is based off of the [Network Time Protocol (NTP)](https://en.wikipedia.org/wiki/Network_Time_Protocol), but is modified to break away from the leader/follower design of NTP.

Including gun in your server/NodeJs projects includes NTS by default. In the browser, you will want to explicitly pull in `gun/nts.js`; i.e. `<script src="https://cdn.jsdelivr.net/npm/gun/nts.js"></script>`

To see how fast and accurate NTS is, visit the [example NTS page](http://gunjs.herokuapp.com/game/nts.html). You may also wish to inspect the [source](https://github.com/amark/gun/blob/master/examples/game/nts.html) of the example.

## Some Caveats
At this time, non-browser nodes do not obey the NTS protocol. Instead, browser nodes converge on the time provided by any server nodes. This is theoretically temporary and will be resolved in the near future to support dynamic multi-peer updates.