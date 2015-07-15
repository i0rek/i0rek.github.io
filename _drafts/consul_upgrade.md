## Upgrading consul infrastructure

Yesterday I upgraded our consul infrastructure. Beforehand I searched for reports from others and didn't find any. That’s the reason I am going to describe what I did so that others who are in a similar situation like I are luckier.
For [Wunderlist](https://www.wunderlist.com) we are using consuls key value store, node statuses and some service discovery. We are running 3 servers and ~700 clients. Our old version was 0.4.1 and I wanted to upgrade to 0.5.2. You should head to [upgrading consul](https://consul.io/docs/upgrading.html) and read everything carefully - it is a good start.

### Preparation

[Upgrade specifics](https://consul.io/docs/upgrade-specific.html) mentions three possible problems: the ACL system changes, the different backend store for persisting the Raft log, and the introduction of tombstones. We don't use ACL yet so that’s fine. Since the migration utility for the Raft log is still part of 0.5.2 that is also taken care of. The tombstones are left which can be dealt with in two ways: update the servers in any order within 15minutes or update the followers first and the leader afterwards. I opted for the latter because I don't like to be in a hurry.

### Testing

In order to get a feeling how this is going to work I setup a test environment with 3 v0.4.1 servers and 1 v0.4.1 client. I upgraded the servers and observed what happens from the clients perspective with `consul monitor --log-level=debug`. For the upgrade itself I replaced the consul executable with the one for v0.5.2 and used `kill -9` to stop the running consul. __It is important to not let the server leave the cluster__. In our case consul is managed by upstart and `stop consul` would have issued a leave which is the reason I had to kill it manually. I then waited for it to be restarted. I did that on server after another.
Although there were brief moments where there was no leader, everything got back to normal pretty fast. The client noticed the changes in the cluster, but was able to deal with it without me doing anything. After the server upgrade, I had 1 v0.4.1 client and 3 0.5.2 servers talking to each other.
I repeated this procedure one more time in another test environment and everything was good the second time too.

### Do it live!

Repeating the procedure a third time in production was way more scary but as boring. The upgrade of our production servers worked fine. The clients will be upgraded as we spin up new servers. I don't mind having old clients as long as I don't need any of the new features for them.

### Conclusion

Since consul is an integral part of our infrastructure upgrading is necessary and scary at the same time. By preparing and testing the upgrade we were able to pull it off without any problems. Upgrading is important and if it is scary you should do it none the less.
