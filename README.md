[![Travis branch](https://img.shields.io/travis/amine7536/goqonto/v1.svg?style=flat-square)](https://travis-ci.org/amine7536/goqonto)

# GoQonto
Qonto API (v1) Go client

## Installation

The import path for the package is gopkg.in/amine7536/goqonto.v1

To install it, run:

```
go get gopkg.in/amine7536/goqonto.v1
```

## API documentation

Qonto API v1 documentation is located at : https://api-doc.qonto.eu/1.0/welcome


## Usage

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"os"

	"gopkg.in/amine7536/goqonto.v1"
)

type AuthTransport struct {
	*http.Transport
	Slug   string
	Secret string
}

func (t AuthTransport) RoundTrip(r *http.Request) (*http.Response, error) {
	r.Header.Set("Authorization", fmt.Sprintf("%s:%s", t.Slug, t.Secret))
	return t.Transport.RoundTrip(r)
}

func main() {

	apiURL := os.Getenv("QONTO_API")
	orgID := os.Getenv("QONTO_ORD_ID")
	orgSlug := os.Getenv("QONTO_ORG_SLUG")
	orgSecret := os.Getenv("QONTO_ORG_SECRET")

	client := http.Client{
		Transport: AuthTransport{
			&http.Transport{},
			orgSlug,
			orgSecret,
		},
	}

	qonto := goqonto.New(&client, apiURL)

	ctx := context.Background()

	orga, _, err := qonto.Organizations.Get(ctx, orgID)
	if err != nil {
		log.Fatalln(err.Error())
	}
	prettyPrint(orga)

	params := &goqonto.TransactionsOptions{
		Slug: orga.Slug,
		IBAN: orga.BankAccounts[0].IBAN,
	}

	list := &goqonto.ListOptions{
		Page:    1,
		PerPage: 10,
	}

	transactions, resp, err := qonto.Transactions.List(ctx, params, list)
	if err != nil {
		log.Fatalln(err.Error())
	}

	for _, trx := range transactions {
		prettyPrint(trx)
	}

	prettyPrint(resp.Meta)
}

func prettyPrint(v interface{}) {
	b, _ := json.MarshalIndent(v, "", "  ")
	println(string(b))
}
```

## Credits

This client is heavily inspired by DigitalOcean GoDo : https://github.com/digitalocean/godo
