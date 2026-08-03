# Contribution 3: Rollbar: Rollbar plugin cannot authenticate with Rollbar "v2" project access tokens

**Contribution Number:** 3

**Student:** Rogelio Perez Montero

**Issue:** [backstage/community-plugins#10111](https://github.com/backstage/community-plugins/issues/10111)

**Status:** Not Started (issue selected, no PR opened yet)

---

## Why I Chose This Issue

After finishing my last contribution with Copilot "Invalid Date" fix, I wanted my next issue to stay in the same lane. 
I wanted to work on a API-integration bug in the same repo. I truly enjoy working in this repository
because the maintainers are fast and reliable with comments and support.
I found myself understanding more their code as well. I got to see how they have different versions of their code and find it really interesting to work with new and legacy versions.

---
## Understanding the Issue
 
### Problem Description
 
The Rollbar plugin can't load a project's data if that project uses a newer Rollbar API key. Rollbar calls these "v2" keys. Almost every Rollbar project uses v2 keys now, since Rollbar stopped letting people create the old kind.
 
The issue is that the plugin asks Rollbar for a list of API keys for a project, and picks out the one that's for reading data. With old-style keys, Rollbar's answer includes the actual secret value of the key. With v2 keys, Rollbar's answer leaves that value out completely. Rollbar only shows you a v2 key's value one time, right when you first create it.
 
So when the plugin looks for that secret value on a v2 key, it isn't there. The plugin sees nothing, assumes there's no valid key at all, and gives up with an error instead of loading the data.
 
### Expected Behavior
 
The plugin should be able to load a project's data whenever that project has a working "read" key, whether that key is the old style or the new v2 style.
 
### Current Behavior
 
When a project only has v2 keys, which is true for basically every project now, the plugin can't find the key's value and throws an error instead of showing the data. The page fails to load.
 
### Affected Components
 
- **Backend only.** The bug lives in one function, `getProjectMetadata()`, inside the file `workspaces/rollbar/plugins/rollbar-backend/src/api/RollbarApi.ts`. This function is the part of the code responsible for asking Rollbar which API key to use.
