Timeline of Exploit:

 - Internally detected June 2018.
 - Decision was to fix prior to launch of any payment system.
 - December 2018 FrontPageGuardian of notabug.io discovered the attack.
 - go1dfish notified team of attack.
 - Less than 1 week later, fix published.

Big thanks to FrontPageGuardian and go1dfish for their help and keeping us accountable during this time. More information on the migration and upgrade below.

The gist is.... Base64 by default, and if that errors like @bugs181 proposed it'll try UTF8, so old data readable all new data Base64 so seamless upgrade path (will only break if other peers if they get new data and they have not upgraded), shuffle attack will allow any old SEA formatted data to validate if it was updated before Dec 31 2018 (adjustable by you `SEA.opt.shuffle_attack` timestamp number, `SEA.opt.unpack` handles format switching) if new format fails to match, new format should have newer vectors and therefore take precedence. Not sure compatibility implications after Dec 31 2018 though. If somebody logs in on an account old format it'll work (but maybe not after Dec 31 2018 if not already upgraded, adjustable by you), and then attempt to automatically do a "change password" (using current password) that will re-save account data thus hopefully upgrading login/core-account info to new format (but nothing else, as I have no knowledge of other data, unless I start crawling and performing writes on every read, which would be bad), and I almost have logins on new format working correctly also. But keep in mind, any old peers will reject upgraded/migrated data as invalid, so there are easy breaking points. Migrating the rest of the data is probably also a good idea, but I'm not sure whether that'll be better than just letting it be hopefully working with backwards compatibility even if exposed to shuffle attacks.

The default is to allow shuffle attack on all old formatted data prior to Dec 31 2018 since it is already vulnerable but at least this will mean old data validates and is still accessible. If you do not want this, you can set `SEA.opt.shuffle_attack = 0` (0 is the timestamp for UTC time start/origin) and then SEA will error that the data is invalid.

Old format data should still validate after end of Dec 31 2018 as long as that data is from before end of Dec 31 2018.
The assumption is all peers will have upgraded to this fix by Dec 31 2018, so please upgrade.

So any/all new data should be in new format and anybody creating a shuffle attack in future either:

 - can only shuffle old data (but this is already the case, so it is no stronger/weaker than before)
 - if any person updates their old data, then attacker can't shuffle it since old format < new format
 - an attacker can't shuffle data after Dec 31 2018 unless they say it is old data but then old data < new data

We are leaving data migration to dApp developers because if we did it automatically it'd be a recursive graph crawling scary nightmare since we don't know their schema as well as they do, but we do know the schema on account data (since SEA generates that) and can auto-migrate it.

 > Note though, other peers running old code will reject new code's attempt to auto migrate account data.

Good news is that is idempotent, so... each time it fails, on every new login, it'll be like "this is old format" and attempt to auto migrate, until finally other peers will have updated. Then from that point onward, it won't see any difference in format, and therefore stop trying to auto migrate cause it doesn't need to (however, even if it tried, it wouldn't hurt anything).