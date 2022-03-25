main.md

# keyword untuk concurency
1. go
2. <- 
3. chan

coba cari 3 hal ini
1. \bgo\b
2. \b<-\b
3. \b chan\b

# butuh contoh real-case of concurrency?
setelah membuka gin, bisa disimpulkan: concurrency banyak digunakan saat testing.



# example concurrenct case in gin
## 1. <-clientGone (channel)
stream response gimana?

``
// Stream sends a streaming response and returns a boolean
// indicates "Is client disconnected in middle of stream"
func (c *Context) Stream(step func(w io.Writer) bool) bool {
	w := c.Writer
	clientGone := w.CloseNotify()
	for {
		select {
		case <-clientGone:
			return true
		default:
			keepOpen := step(w)
			w.Flush()
			if !keepOpen {
				return false
			}
		}
	}
}
``

## 2.  <-chan bool
``
// CloseNotify implements the http.CloseNotify interface.
func (w *responseWriter) CloseNotify() <-chan bool {
	return w.ResponseWriter.(http.CloseNotifier).CloseNotify()
}
``

return nya `<-chan` ?

sebenarnya banyak hasil pencarian terkait <-chan
tetapi digunakannya untuk pengujian 

## 3.  go readWriteKeys(c.Copy())

```
func TestRaceContextCopy(t *testing.T) {
	DefaultWriter = os.Stdout
	router := Default()
	router.GET("/test/copy/race", func(c *Context) {
		c.Set("1", 0)
		c.Set("2", 0)

		// Sending a copy of the Context to two separate routines
		go readWriteKeys(c.Copy())
		go readWriteKeys(c.Copy())
		c.String(http.StatusOK, "run OK, no panics")
	})
	w := PerformRequest(router, http.MethodGet, "/test/copy/race")
	assert.Equal(t, "run OK, no panics", w.Body.String())
}

func readWriteKeys(c *Context) {
	for {
		c.Set("1", rand.Int())
		c.Set("2", c.Value("1"))
	}
}
```

## 4. make(chan int)

```
func TestRenderIndentedJSONPanics(t *testing.T) {
	w := httptest.NewRecorder()
	data := make(chan int)

	// json: unsupported type: chan int
	err := (IndentedJSON{data}).Render(w)
	assert.Error(t, err)
}
```

## 5. make(chan int)
```
// Done returns nil (chan which will wait forever) when c.Request has no Context.
func (c *Context) Done() <-chan struct{} {
	if c.Request == nil || c.Request.Context() == nil {
		return nil
	}
	return c.Request.Context().Done()
}
```

nanti lanjut belajar lagi besok!