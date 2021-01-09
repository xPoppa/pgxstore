# pgxstore

A session store backend for [gorilla/sessions](http://www.gorillatoolkit.org/pkg/sessions) - [src](https://github.com/gorilla/sessions).

Forked from [antonlindstrom/pgstore](https://github.com/antonlindstrom/pgstore) and modified to use [jackc/pgx](https://github.com/jackc/pgx) instead of database/sql.

From the [pgx docs](https://github.com/jackc/pgx#choosing-between-the-pgx-and-databasesql-interfaces):

> It is recommended to use the pgx interface if:
>
> 1. The application only targets PostgreSQL.
> 2. No other libraries that require database/sql are in use.

## Installation

    go get github.com/yi-jiayu/pgxstore

## Documentation

Available on [godoc.org](http://www.godoc.org/github.com/yi-jiayu/pgxstore).

See http://www.gorillatoolkit.org/pkg/sessions for full documentation on underlying interface.

### Example

```go
package examples

import (
	"log"
	"net/http"
	"time"

	"github.com/yi-jiayu/pgxstore"
)

// ExampleHandler is an example that displays the usage of PGStore.
func ExampleHandler(w http.ResponseWriter, r *http.Request) {
	// Fetch new store.
	store, err := pgstore.NewPGStore("postgres://user:password@127.0.0.1:5432/database?sslmode=verify-full", []byte("secret-key"))
	if err != nil {
		log.Fatalf(err.Error())
	}
	defer store.Close()

	// Run a background goroutine to clean up expired sessions from the database.
	defer store.StopCleanup(store.Cleanup(time.Minute * 5))

	// Get a session.
	session, err := store.Get(r, "session-key")
	if err != nil {
		log.Fatalf(err.Error())
	}

	// Add a value.
	session.Values["foo"] = "bar"

	// Save.
	if err = session.Save(r, w); err != nil {
		log.Fatalf("Error saving session: %v", err)
	}

	// Delete session.
	session.Options.MaxAge = -1
	if err = session.Save(r, w); err != nil {
		log.Fatalf("Error saving session: %v", err)
	}
}
```
