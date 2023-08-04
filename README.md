# Chroma Go

A simple Chroma Vector Database client written in go

| Note: This is my first code in go so don't judge me for newb mistakes


## Feature Parity with ChromaDB API

- [x] Reset
- [x] Heartbeat
- [ ] List Collections
- [ ] Raw SQL
- [ ] Get Version
- [x] Create Collection
- [ ] Delete Collection
- [x] Collection Add
- [x] Collection Get (partial without additional parameters)
- [x] Collection Count
- [x] Collection Query
- [ ] Collection Modify
- [ ] Collection Update
- [ ] Collection Upsert
- [ ] Collection Create Index
- [ ] Collection Delete - delete documents in collection


## Installation

```bash
go get github.com/amikos-tech/chroma-go
```

or:

```go
import (
    chroma "github.com/amikos-tech/chroma-go"
)
```

## Usage


Ensure you have a running instance of Chroma running. We recommend one of the two following options:

- Official documentation - https://docs.trychroma.com/usage-guide#running-chroma-in-clientserver-mode
- If you are a fan of Kubernetes, you can use the Helm chart - https://github.com/amikos-tech/chromadb-chart (Note: You
  will need `Docker`, `minikube` and `kubectl` installed)


**The Setup (Cloud-native):**

```bash
minikube start --profile chromaj
minikube profile chromaj
helm repo add chroma https://amikos-tech.github.io/chromadb-chart/
helm repo update
helm install chroma chroma/chromadb --set chromadb.allowReset=true,chromadb.apiVersion=0.4.4
```


Consider the following example where:

- We create a new collection
- Add documents using OpenAI embedding function
- Query the collection using the same embedding function

```go
package main

import (
	"fmt"
	chroma "github.com/amikos-tech/chroma-go"
	openai "github.com/amikos-tech/chroma-go/openai"
	godotenv "github.com/joho/godotenv"
	"os"
)
func main(){
	client := chroma.NewClient("http://localhost:8000")
	collectionName := "test-collection"
	metadata := map[string]string{}
	err := godotenv.Load(".env")
	if err != nil {
		fmt.Printf("Error loading .env file: %s", err)
		return
	}
	embeddingFunction := openai.NewOpenAIEmbeddingFunction(os.Getenv("OPENAI_API_KEY")) //create a new OpenAI Embedding function
	distanceFunction := chroma.L2
	_, errRest := client.Reset() //reset the database
	if errRest != nil {
		fmt.Printf("Error resetting database: %s", errRest)
		return
	}
	col, err := client.CreateCollection(collectionName, metadata, true, embeddingFunction, distanceFunction)
	documents := []string{
		"This is a document about cats. Cats are great.",
		"this is a document about dogs. Dogs are great.",
	}
	ids := []string{
		"ID1",
		"ID2",
	}

	metadatas := []map[string]string{
		{"key1": "value1"},
		{"key2": "value2"},
	}
	_, addError := col.Add(nil, metadatas, documents, ids)
	if addError != nil {
		fmt.Printf("Error adding documents: %s", addError)
        return 
    }
	countDocs, qrerr := col.Count()
	if qrerr != nil {
		fmt.Printf("Error counting documents: %s", qrerr)
        return
	}
	fmt.Printf("countDocs: %v\n", countDocs) //this should result in 2
	qr, qrerr := col.Query([]string{"I love dogs"}, 5, nil, nil, nil)
	if qrerr != nil {
        fmt.Printf("Error querying documents: %s", qrerr)
        return
    }
	fmt.Printf("qr: %v\n", qr.Documents[0][0]) //this should result in the document about dogs
}
```

## References

- https://docs.trychroma.com/ - Official Chroma documentation
- https://github.com/amikos-tech/chromadb-chart - Chroma Helm chart for cloud-native deployments
