GUN generally considers sharding as *not a good thing.

*With the exception that you are running your own physical machines. Let's talk about that and lay out the landscape.

### 1. Database as a Service
Such as Compose, Heroku's Postgres, Dropbox Datastore, Firebase, Parse, etc.

### 2. Deployed Database in the Cloud
This is where you spin up a box on AWS EC2, Google Cloud Engine, Microsoft Azure, Digital Ocean, etc. and install and maintain your own database - such as MySQL, MongoDB, RethinkDB, etc.

### 3. On your own Physical Machine
While similar to (2), you ultimately own the hardware which you do deployments on - where when resources run low, you have to physically add hard drives.

## Advantages

The advantage of (1) is that (2) and (3) may be hard and difficult, which is why DevOps and SysAdmins are critical roles. Database as a Services outsource these roles for you with the downside of typically being expensive when you surpass the freemium tier, slower network latency, and not owning your data.

The advantage of (3) is that you own your data, which for security or regulatory concerns might be necessary. It suffers a serious pain point though, upfront costs and manual intervention as well as extensive knowledge across disciplines.

Deploying to the cloud is the middle ground, you are paying to outsource the physical work and take advantage of provisioning as many virtual machines as possible. The important distinction is that things are virtualized, whether that be the computation or the storage - meaning the cloud provider has abstracted the scaling of these resources for you.

## Observations

Traditionally database servers had to be provisioned, acting as a transactor for data on one or more disks. To be precise, this transactor is an abstraction and virtualization layer for storing structured data. That way the application developer does not have to worry about storage location. But cloud providers have already virtualized storage, such as S3 or GCS or Azure Storage which are "infinite" hard drives. This is not to say that fixed size allocated disks are unavailable, they are - and significantly faster, but the constraints on their size are arbitrarily limited by the underlying virtualization.

These arbitrary constraints are the very things that the database and cloud are trying to remove. Yet because there is such a prevalent expectation that things are limited, people keep resorting to finite drives by default. And how do you get around those defaults? By chunking data into tiny fixed sizes and storing and replicating them on a growing set of hard drives that are dedicated to a specific "shard" key or range. This is when sharding is necessary, such as when you run your own physical machines.

## Misconception

Ironically, despite the scalability of the cloud's storage, people still deploy and shard databases across finite drives on these providers. This results in sharding still being a common practice, but has the following two unfortunate disadvantages with only one advantage:

 - **A.** You pay for compute time as well as the total allocated storage cost rather than storage used.
 - **B.** You still need devops and sysadmin expertise to configure and coordinate shard keys and boxes.
 - **C.** With the advantage of not getting locked into any proprietary virtualization system.

Option (C) alone is worthwhile, but this is not a reason to suddenly start sharding on the cloud, it would be a reason to move off the cloud entirely to (3) because the underlying provisioning of machines is proprietary as well. So what? Maybe you are using the cloud to optimize cost, in which case (A) is going to cost more anyways. So no matter how you approach it, attempting to shard in the cloud is a misconception and should be avoided.

## Conclusion

Therefore GUN does not recommend sharding, as it assumes the majority of users will be deploying in the cloud. Additionally, this then removes one massive pain point for the developer - letting GUN automatically cache based on dynamic load rather than storage constraints. This makes things faster for your end user and less work for you personally.

Finally, if you want to use GUN on your own machines (3) where you will have to shard, it is still possible via the priority and no-expiry options (documentation coming). Therefore GUN can shard when physically necessary, but it is highly recommended to be avoided everywhere else in order to reduce costs and complexity.