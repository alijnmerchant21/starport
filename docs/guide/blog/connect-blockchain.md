---
description: Blockchain Client in Go
order: 2
---

# Creating a Blockchain Client in Go

Learn how to connect your Blockchain to an independent application with RPC.

After creating the blog blockchain in this tutorial you will learn how to connect to your blockchain from a seperate client.

## Use the Blog Blockchain

Navigate to a separate directory right next to the Blog Blockchain you just built in the last chapter.

## Creating a Blockchain Client

Create a new directory called `blogclient` on the level next to `blog` directory. As the name suggests, `blogclient` will contain a standalone Go program that will act as a client to your `blog` blockchain.

The command

```bash
ls
```

should show you just `blog` now, or least be listed when you have more directories here.

Create your blogclient directory first, change your current working directory into it and initialize a new go module.

```bash
mkdir blogclient
cd blogclient
go mod init github.com/cosmonaut/blogclient
```

You will end up with the go.mod file inside your `blogclient` directory.

Create the main.go file and add the content as follows.

Your blockchain client has only two dependencies: the `blog` blockchain (`types` for message types and a query client) and `starport` (for the `cosmosclient` blockchain client).

```go
module github.com/cosmonaut/blogclient

go 1.17

require (
	github.com/cosmonaut/blog v0.0.0-00010101000000-000000000000
	github.com/tendermint/starport v0.18.0
)

replace github.com/cosmonaut/blog => ../blog
replace github.com/gogo/protobuf => github.com/regen-network/protobuf v1.3.3-alpha.regen.1
```

The `replace` directive uses the package from the local `blog` directory (specified as a relative path).

Cosmos SDK uses a custom version of the `protobuf` package, so use the `replace` directive to specify the correct dependency.

`blogclient` will eventually have only two files: `main.go` for the main logic of the client and `go.mod` for specifying dependencies.

### Main Logic of the Client in main.go

Add the following code to your main.go file to make a connection to your blockchain from a separate app.

```go
package main

import (
	"context"
	"fmt"
	"log"

	// importing the types package of your blog blockchain
	"github.com/cosmonaut/blog/x/blog/types"
	// importing the general purpose Cosmos blockchain client
	"github.com/tendermint/starport/starport/pkg/cosmosclient"
)

func main() {

	// create an instance of cosmosclient
	cosmos, err := cosmosclient.New(context.Background())
	if err != nil {
		log.Fatal(err)
	}

	// account `alice` was initialized during `starport chain serve`
	accountName := "alice"

	// get account from the keyring by account name and return a bech32 address
	address, err := cosmos.Address(accountName)
	if err != nil {
		log.Fatal(err)
	}

	// define a message to create a post
	msg := &types.MsgCreatePost{
		Creator: address.String(),
		Title:   "Hello!",
		Body:    "This is the first post",
	}

	// broadcast a transaction from account `alice` with the message to create a post
	// store response in txResp
	txResp, err := cosmos.BroadcastTx(accountName, msg)
	if err != nil {
		log.Fatal(err)
	}

	// print response from broadcasting a transaction
	fmt.Print("MsgCreatePost:\n\n")
	fmt.Println(txResp)

	// instantiate a query client for your `blog` blockchain
	queryClient := types.NewQueryClient(cosmos.Context)

	// query the blockchain using the client's `PostAll` method to get all posts
	// store all posts in queryResp
	queryResp, err := queryClient.Posts(context.Background(), &types.QueryPostsRequest{})
	if err != nil {
		log.Fatal(err)
	}

	// print response from querying all the posts
	fmt.Print("\n\nAll posts:\n\n")
	fmt.Println(queryResp)
}
```

Read the comments in the code carefully to learn details about each line of code.

There is more to learn about the `cosmosclient` package used here.
[Jump to the `cosmosclient` reference](https://pkg.go.dev/github.com/tendermint/starport/starport/pkg/cosmosclient) to learn how to use the Client with Options and KeyringBackend.

## Running the Blockchain and the Client

Make sure your blog blockchain is still running with `starport chain serve`.

Install dependencies for your `blogclient`:

```bash
cd blogclient

go mod tidy
```

Run the blockchain client:

```bash
go run main.go
```

If successful, the results of running the command are printed to the terminal:

```bash
go run main.go
# github.com/keybase/go-keychain
### Some warnings might be displayed which can be ignored
MsgCreatePost:

Response:
  Height: 3222
  TxHash: AFCA76B0FEE5113382C068967B610180C105FCE045FF8C7943EA45EF4B7A1E69
  Data: 0A280A222F636F736D6F6E6175742E626C6F672E626C6F672E4D7367437265617465506F737412020801
  Raw Log: [{"events":[{"type":"message","attributes":[{"key":"action","value":"CreatePost"}]}]}]
  Logs: [{"events":[{"type":"message","attributes":[{"key":"action","value":"CreatePost"}]}]}]
  GasWanted: 300000
  GasUsed: 45805


All posts:

Post:<creator:"cosmos1j8d8pyjr5vynjvcq7xgzme0ny6ha30rpakxk3n" title:"foo" body:"bar" > Post:<creator:"cosmos1j8d8pyjr5vynjvcq7xgzme0ny6ha30rpakxk3n" id:1 title:"Hello!" body:"This is the first post" > pagination:<total:2 > 
```

You can confirm the new post with using the `blogd query blog posts` command learned in the previous chapter.
The result should look similar to

```bash
Post:
- body: bar
  creator: cosmos1j8d8pyjr5vynjvcq7xgzme0ny6ha30rpakxk3n
  id: "0"
  title: foo
- body: This is the first post
  creator: cosmos1j8d8pyjr5vynjvcq7xgzme0ny6ha30rpakxk3n
  id: "1"
  title: Hello!
pagination:
  next_key: null
  total: "2"
```

Congratulations, you have just created a post using a seperate app.

When you publish your blockchain project to GitHub, you won't need to use the replace function in your `go.mod` file anymore and can directly use your GitHub repository to fetch the types and interact with your program.