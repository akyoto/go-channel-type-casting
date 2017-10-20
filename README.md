# Type-casting interface{} to chan interface{}

This is an experience report for the Go programming language.

## Problem

Let's say you're building a web application and offering a generic search for your online database:

![DB search](https://puu.sh/y2CCe/fe41c17835.png)

The database API creates a channel of correctly typed objects via reflection:

```go
// All returns a stream of all objects in the given table.
func (db *Database) All(table string) (interface{}, error) {
	channel := reflect.MakeChan(reflect.ChanOf(reflect.BothDir, reflect.PtrTo(db.types[table])), 0).Interface()
	err := db.Scan(table, channel)
	return channel, err
}
```

As the web developer, I am now calling this function in the route that performs the DB search:

```go
stream, err := DB.All(dataTypeName)
```

`stream` is of type `interface{}` at this point. However, I can't do a `for range` loop over this stream.

My first attempt, which seemed logical to me, was casting the type to a generic channel `chan interface{}`.

```go
for obj := range stream.(chan interface{}) {
	process(obj)
}
```

At runtime, I get the following error:

```
interface conversion: interface {} is chan *User, not chan interface {}
```

This means you can only type-cast the channel to the 100% correct type, not a slightly more generic version of the type. You can type-cast it to `chan *User` but `chan interface{}` doesn't work.

The only workaround I could think of is writing unmaintainable code using `switch` / `case` statements:

```go
switch dataTypeName {
	case "Analytics":
		for obj := range stream.(chan *Analytics) {
			process(obj)
		}
	case "Group":
		for obj := range stream.(chan *Group) {
			process(obj)
		}
	case "Post":
		for obj := range stream.(chan *Post) {
			process(obj)
		}
	case "Settings":
		for obj := range stream.(chan *Settings) {
			process(obj)
		}
	case "SoundTrack":
		for obj := range stream.(chan *SoundTrack) {
			process(obj)
		}
	case "Thread":
		for obj := range stream.(chan *Thread) {
			process(obj)
		}
	case "User":
		for obj := range stream.(chan *User) {
			process(obj)
		}
	}
```

This is of course far from an ideal solution. I am keeping this example short, in reality there are far more data types than listed here. Every time I add a new datatype, I have to update this `switch`/`case` statement. In short, it's a maintenance horror.
