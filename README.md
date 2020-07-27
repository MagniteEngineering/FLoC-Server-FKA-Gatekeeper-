# Gatekeeper

## An Actional Proposal for Gatekeeper in Cohort Creation 

### Introduction

Today, online advertising systems rely on 3rd party cookies to store the cross-publisher IDs  to group users into audience segments or cohorts using sophisticated algorithms based on their browsing behavior. Marketers use these cohorts to match advertising content to the users who are likely to be the most receptive. They are then able to record which users, and cohorts, generated better marketer success metrics (e.g., visited the marketer website, purchased the marketer’s product) to determine how to adjust their budgeting and targeting of future content.

The ultimate goal of this system is to match marketing messages to user interests. Users get to see more relevant ads, marketers get to reach a more receptive audience, and publishers get to more effectively monetize their content while lowering ad loads.  

However, the use of individual user ids across page domains creates numerous privacy problems, most notably the lack of relationship between the marketer and the user, specifically for prospects. As a result, there have been numerous proposals to eliminate the practice of sending identifiers from sellers to buyers. However, these proposals often self-contain the process of creating cohorts or groups entirely within a centralized organization, which is problematic for several reasons:

* browsers are not aware of each organizations relationship with each person nor the content matching and analysis use cases every organization relies on to operate their business 
* some browser vendors also operate advertising businesses, app stores, and thus have conflicts of interest 
* no system can be expected to work properly when the entirety of decision making is concentrated in the hands of roughly three companies
 
As such, we believe we need a system where more than one trusted entities have the ability to create cohorts and, under strictly limited and governed rule sets, process identifiers for the sole purpose of creating these cohorts. This proposal attempts to provide a solution that will enable proprietary grouping of users into cohorts based on their browsing behavior in a way that prevents identifying individuals, doesn’t share their browsing behavior with marketers and protects user interests and buying habits from being shared with publishers -- but also avoids fully concentrating all decisioning in the hands of single points of failure. The general philosophy behind this paper is that if a process is ok to be carried out in the browser, it should also be allowed by the publisher community as long as this is very clearly defined, limited, and governed to ensure proper user privacy and choice..

### Gatekeeper
A “gatekeeper” is designed as a service to create cohorts from limited, defined user identifiers, and will live in a server side construct outside of the browser. It must be either a not for profit entity, some other limited entity with strict controls, or a browser. It should not be a for-profit entity, and the total number of gatekeepers should be strictly limited. The gatekeepers’ role is to receive common ids from members of a 1st party set and group  those users into cohorts.  The gatekeeper will then return an individual user ids cohort membership list to the publisher.  The response can be cached in the browser, or requested in real time during an ad impression event. 
### Federated Set
A collection of websites can agree to share a common user id between themselves for the purpose of building a cohort.  To ensure a level playing field and allow smaller publishers to more effectively compete with larger rivals, websites are not required to be owned by the same entity although they may be.  Members of a Federated set must publish their membership list at a well known endpoint.  Browsers should then allow members of the set to pass a common user id set in a browser storage container between themselves.
### Business rules
1. Domains may elect to federate themselves together into a Federated set operated by a single gatekeeper
2. A domain may be a member of 1 first-party sets
3. The set decides which gatekeeper to use
4. Gatekeepers will assign the each domain in the set a unique id, that id may be passed from domain to gatekeeper along with TLD+1 & contextual information
5. Members of the set may not see one anothers userid, only the gatekeeper can un salt the userid
6. Publishers must declare their set membership in the header
7. Publishers must declare their gatekeeper in the header
8. Gatekeeper will log all inbound requests and outbound responses for 24 hrs. 
### API Example Flow

A website (freeknitting.com) who chooses to utilize gatekeeper1 and is a member of the Prebid set can add 2 new html tags to the page header:

```
<link rel="set-import" href="https://prebid.org/set/membership.json">
<link rel="gatekeeper-import" href="https://gatekeeper1.com">
```

Supporting browsers would read the tags and execute a request first to nba-set.com/set/membership.json?domain=lakers.com file to verify that the TLD+1 is a member, 

```
member=true
```

then to the gatekeeper, if the domain does not have a userID in LS the gatekeeper will return one. 

```
{
   "userID": 123
}
```

The domain may then make a second request to the gatekeeper for the set ids cohort membership like this gatekeeper1.com?userID=123&contextual=basketball&domain=lakers.com, where 123 is a unique identifier generated by the gatekeeper, cached in the browser, then included on all cohort requests from this domain.  The gatekeeper will record the contextual and TLD+1 information in the cohort request and cluster users with similar behavior.  The gatekeeper will return the cohort ids, label, and contextual information associated with the cohort id.

``` 
{
   "cohortid":ad45,
   "cohortlabel":"NBA Enthusiasts",
   "context":"Basketball"
}
```

The browser would store this cohortId and make it available to the publisher via a JS API.

```
navigator.getCohortId();
```

Publishers could use this Cohort ID for analytics purposes and forward it to marketers in ad requests. marketers could use this Cohort ID to identify which cohorts are engaging with their brand and could target ads to these users in this cohort.

This system would provide for the needed advertising functionality of both publishers and buyers while protecting users’ privacy.

### Use Cases

#### Assumptions

1. Kings and Lakers are in the federated set. 
2. set membership verification returns TRUE (not shown)
3. the set uses Gatekeeper 1

#### GateKeeper Declaration (new user)

1. User clears Gatekeeper id and loads Kings.com
2. Browser verifies domain is a member of declared set
3. GateKeeper declaration - browser sends domain, and contextual information about the site to gatekeeper 
4. Gatekeeper sees that this browser+domain combo does not have an id so generates a domain specific value and returns it to the domain
5. Logs inbound request & outbound response
6. User navigates to lakers.com
7. Lakers.com user id was cleared in step 1
8. Browser verifies domain is a member of declared set
9. GateKeeper declaration - browser sends domain, and contextual information about the site to gatekeeper
10. Gatekeeper sees that this browser+domain combo does not have an id so generates a domain specific value and returns it to the domain
11. Logs inbound request & outbound response

![GateKeeper Declaration (new user)](https://user-images.githubusercontent.com/14223042/87727366-14621380-c77e-11ea-88d1-ee23a8578b10.png)

#### GateKeeper Declaration (existing user)

1. User loads Kings.com
2. Browser verifies domain is a member of declared set
3. GateKeeper declaration - browser sends userID, domain, and contextual information about the site to gatekeeper 
4. Gatekeeper logs inbound request & outbound response
5. Gatekeeper adds request data to cohort creation ML
6. User navigates to lakers.com
7. Browser verifies domain is a member of declared set
8. GateKeeper declaration - browser sends userID, domain, and contextual information about the site to gatekeeper
9. Gatekeeper logs inbound request & outbound response
10. Gatekeeper adds request data to cohort creation ML

![GateKeeper Declaration (existing user)](https://user-images.githubusercontent.com/14223042/87727362-12985000-c77e-11ea-8e43-3d78544fa3c8.png)

#### Cohort Creation

1. User loads Kings.com
2. GateKeeper declaration - browser sends userID, domain, and contextual information about the site to gatekeeper 
3. Gatekeeper un salts userId
4. Gatekeeper adds user + data to cohort creation ML

![Cohort Creation](https://user-images.githubusercontent.com/14223042/87728431-4bd1bf80-c780-11ea-8b68-b4108e78666c.png)

#### Cohort Retrieval

1. User  loads Kings.com
2. Browser verifies domain is a member of declared set
3. GateKeeper declaration - browser sends userID, domain, and contextual information about the site to gatekeeper 
4. Gatekeeper un salts userId
5. Gatekeeper queries cohort store w/ unsalted user 
6. Gatekeeper returns cohort membership list


![Cohort Retrieval](https://user-images.githubusercontent.com/14223042/87727344-0dd39c00-c77e-11ea-93c1-2f56b4a60174.png)

Notes:
Users will be able to see which gatekeepers have data on them, and what cohorts they are a part of.
