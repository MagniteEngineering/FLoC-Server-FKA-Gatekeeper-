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
### First Party Set
A collection of websites can agree to share a common user id between themselves for the purpose of building a cohort.  To ensure a level playing field and allow smaller publishers to more effectively compete with larger rivals, websites are not required to be owned by the same entity although they may be.  Members of a first party set must publish their membership list at a well known endpoint.  Browsers should then allow members of the set to pass a common user id set in a browser storage container between themselves.
### Business rules
1. Domains may elect to federate themselves together into a first party set operated by a single gatekeeper
2. A domain may be a member of 1 first-party sets
3. The set decides which gatekeeper to use
4. Gatekeepers will assign the browser a unique id, that id may be passed from domain to gatekeeper along with TLD+1 & contextual information
5. Once federated, the members of a set may share the unique id with one another & to a gatekeeper for cohort creation
6. Publishers must declare their set membership in the header
7. Publishers must declare their gatekeeper in the header
### API Example Flow

A website (freeknitting.com) who chooses to utilize gatekeeper1 and is a member of the Prebid set can add 2 new html tags to the page header:

```
<link rel="set-import" href="https://prebid.org/set/membership.json">
<link rel="gatekeeper-import" href="https://gatekeeper1.com">
```

Supporting browsers would read the tags and execute a request first to prebid.org/set/membership.json?domain=freeknitting.com file to verify that the TLD+1 is a member, 

member=true

then to the gatekeeper, if the browser does not have a set id in LS the gatekeeper will return one. 

```
{
   "setUserID": 123
}
```

The domain may then make a second request to the gatekeeper for the set ids cohort membership like this gatekeeper1.com?setUserID=123&contextual=knitting&domain=freeknitting.com, where 123 is a unique identifier generated by the gatekeeper, cached in the browser, then included on all cohort requests from this browser across members of the first party set to this gatekeeper.  The gatekeeper will record the contextual and TLD+1 information in the cohort request and cluster set Ids with similar behavior.  The gatekeeper will return the cohort ids, label, and contextual information associated with the cohort id.

``` 
{
   "cohortid":ad45,
   "cohortlabel":"DIY Enthusiasts",
   "context":"knitting"
}
```

The browser would store this cohortId and make it available to the publisher via a JS API.

```
navigator.getCohortId();
```

Publishers could use this Cohort ID for analytics purposes and forward it to marketers in ad requests. marketers could use this Cohort ID to identify which cohorts are engaging with their brand and could target ads to these users in this cohort.

This system would provide for the needed advertising functionality of both publishers and buyers while protecting users’ privacy.

### Use Case
#### Cohort creation
1. First Party Set Declaration
  1.1 - Prebid set
  1.2 - Hearst set
  1.3 - NBA set
2. Publisher Decelerations
  2.1 publisher declares the gatekeepers they intend to use
  2.2 publisher declares the 1st party sets they are a member of
3. Cohort Exchange
  3.1 - Request
  3.2 - Cohort Response
4. Gatekeeper ML cohort creation
5. Id boundary
6. Publishers pass cohort information to Ad Exchange

![Image of Cohorts](https://user-images.githubusercontent.com/14223042/87160801-c7170b00-c280-11ea-8354-845426fa0188.png)

![cohorts](https://user-images.githubusercontent.com/14223042/87161230-7358f180-c281-11ea-8577-aec36a226417.png)

![Exchange](https://user-images.githubusercontent.com/14223042/87161326-9c798200-c281-11ea-8504-18911b463e4e.png)

Notes:
Users will be able to see which gatekeepers have data on them, and what cohorts they are a part of.
