
```bash
# Create the project
mkdir amplify-api-vtl
cd amplify-api-vtl
git init
amplify init
rm .gitignore # So we can track everything
git add .
git commit -m "amplify init"

# add an API
amplify add api
# Single
# Edit schema
# Remove public
# Add IAM auth
# Add IAM private auth to schema
git add .
git commit -m "Add api pre-push"
amplify push --yes
git add .
git commit -m "Add api post"

# Add a function without API access
amplify add function
git add .
git commit -m "Add function pre-push"
amplify push --yes
git add .
git commit -m "Add function post-push"

# Give the function API access
amplify update function # Give API access
git add .
git commit -m "Add permissions pre-push"
amplify push --yes


# ┌──────────┬───────────────────────┬───────────┬───────────────────┐
# │ Category │ Resource name         │ Operation │ Provider plugin   │
# ├──────────┼───────────────────────┼───────────┼───────────────────┤
# │ Function │ amplifyapivtldc8f102f │ Update    │ awscloudformation │
# ├──────────┼───────────────────────┼───────────┼───────────────────┤
# │ Api      │ amplifyapivtl         │ No Change │ awscloudformation │
# └──────────┴───────────────────────┴───────────┴───────────────────┘

*Note no change for the API*

git add .
git commit -m "Add permissions post-push"
# Note on disk VTL has been updated
```

```diff
diff --git a/amplify/backend/api/amplifyapivtl/build/resolvers/Query.getTodo.auth.1.req.vtl b/amplify/backend/api/amplifyapivtl/build/resolvers/Query.getTodo.auth.1.req.vtl
index 477623b..81c0d69 100644
--- a/amplify/backend/api/amplifyapivtl/build/resolvers/Query.getTodo.auth.1.req.vtl
+++ b/amplify/backend/api/amplifyapivtl/build/resolvers/Query.getTodo.auth.1.req.vtl
@@ -3,6 +3,12 @@ $util.qr($ctx.stash.put("hasAuth", true))
 #set( $isAuthorized = false )
 #set( $primaryFieldMap = {} )
 #if( $util.authType() == "IAM Authorization" )
+  #set( $adminRoles = ["amplifyapivtldc8f102f-dev"] )
+  #foreach( $adminRole in $adminRoles )
+    #if( $ctx.identity.userArn.contains($adminRole) && $ctx.identity.userArn != $ctx.stash.authRole && $ctx.identity.userArn != $ctx.stash.unauthRole )
+      #return($util.toJson({}))
+    #end
+  #end
   #if( !$isAuthorized )
     #if( $ctx.identity.userArn == $ctx.stash.unauthRole )
       #set( $isAuthorized = true )
```

```bash
# Pull the remote VTL
AWS_PROFILE="inodes" aws appsync list-functions --api-id q6l2uhsysrfppbovgxdjnxg7tu > vtl.json
```

```json
{
  "functionId": "3p56doujpzh4nc6qwz5g5r6uvu",
  "functionArn": "arn:aws:appsync:ap-southeast-2:119243932312:apis/q6l2uhsysrfppbovgxdjnxg7tu/functions/3p56doujpzh4nc6qwz5g5r6uvu",
  "name": "QuerygetTodoauth0Function",
  "dataSourceName": "NONE_DS",
  "requestMappingTemplate": "## [Start] Authorization Steps. **
      $util.qr($ctx.stash.put(\"hasAuth\", true))
      #set( $isAuthorized = false )
      #set( $primaryFieldMap = {} )
      #if( $util.authType() == \"IAM Authorization\" )
        #if( !$isAuthorized )
          #if( $ctx.identity.userArn == $ctx.stash.unauthRole )
            #set( $isAuthorized = true )
          #end
        #end
      #end
      #if( !$isAuthorized && $util.isNull($ctx.stash.authFilter) )
      $util.unauthorized()
      #end
      $util.toJson({\"version\":\"2018-05-29\",\"payload\":{}})
      ## [End] Authorization Steps. **
  ",
  "responseMappingTemplate": "$util.toJson({})",
  "functionVersion": "2018-05-29",
  "maxBatchSize": 0
},
```

```bash
# Add dummy value to schema
vi amplify/backend/api/amplifyapivtl/schema.graphql
git add .
git commit -m "Schema change pre push"
amplify push --yes
git add
git commit -m "Schema change post push"

# Pull the remote VTL
AWS_PROFILE="inodes" aws appsync list-functions --api-id q6l2uhsysrfppbovgxdjnxg7tu > vtl-change.json
# Remote now fixed
diff vtl.json vtl-change.json
```

```diff
--- vtl.json	2022-05-20 09:02:43.637994122 +1000
+++ vtl-change.json	2022-05-20 09:38:22.402821924 +1000
@@ -15,7 +15,7 @@
             "functionArn": "arn:aws:appsync:ap-southeast-2:119243932312:apis/q6l2uhsysrfppbovgxdjnxg7tu/functions/3p56doujpzh4nc6qwz5g5r6uvu",
             "name": "QuerygetTodoauth0Function",
             "dataSourceName": "NONE_DS",
-            "requestMappingTemplate": "## [Start] Authorization Steps. **\n$util.qr($ctx.stash.put(\"hasAuth\", true))\n#set( $isAuthorized = false )\n#set( $primaryFieldMap = {} )\n#if( $util.authType() == \"IAM Authorization\" )\n  #if( !$isAuthorized )\n    #if( $ctx.identity.userArn == $ctx.stash.unauthRole )\n      #set( $isAuthorized = true )\n    #end\n  #end\n#end\n#if( !$isAuthorized && $util.isNull($ctx.stash.authFilter) )\n$util.unauthorized()\n#end\n$util.toJson({\"version\":\"2018-05-29\",\"payload\":{}})\n## [End] Authorization Steps. **",
+            "requestMappingTemplate": "## [Start] Authorization Steps. **\n$util.qr($ctx.stash.put(\"hasAuth\", true))\n#set( $isAuthorized = false )\n#set( $primaryFieldMap = {} )\n#if( $util.authType() == \"IAM Authorization\" )\n  #set( $adminRoles = [\"amplifyapivtldc8f102f-dev\"] )\n  #foreach( $adminRole in $adminRoles )\n    #if( $ctx.identity.userArn.contains($adminRole) && $ctx.identity.userArn != $ctx.stash.authRole && $ctx.identity.userArn != $ctx.stash.unauthRole )\n      #return($util.toJson({}))\n    #end\n  #end\n  #if( !$isAuthorized )\n    #if( $ctx.identity.userArn == $ctx.stash.unauthRole )\n      #set( $isAuthorized = true )\n    #end\n  #end\n#end\n#if( !$isAuthorized && $util.isNull($ctx.stash.authFilter) )\n$util.unauthorized()\n#end\n$util.toJson({\"version\":\"2018-05-29\",\"payload\":{}})\n## [End] Authorization Steps. **",
             "responseMappingTemplate": "$util.toJson({})",
             "functionVersion": "2018-05-29",
             "maxBatchSize": 0
```
