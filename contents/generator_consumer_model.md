## Generator and Consumer model
Golang is the right language for such a language model. Because goroutine is designed for such multiple thread/routine to run the job.

And at the same time, golang owns channel struct to make it possible for different threads/routines to communicate with each others.

The following is the simple example to show how to use golang to complement a generator-consumer-model.

### main function
```
const (
    goroutionNum = 100
)

func main(){
	glog.V(3).Infof("[BenchMark]:starting to test")
	startTime := time.Now().UTC().UTC()
	ch := make(chan string, 100)
	done := make(chan bool, goroutionNum)
	for i := 1; i <= goroutionNum; i++ {
		go Consumer(i, ch, done)
	}
	go Producer(ch)
	for i := 1; i <= goroutionNum; i++ {
		<-done
	}

    // analyse the result
	endTime := time.Now().UTC().UTC()
	totalDuration := endTime.Sub(startTime).Minutes()
	glog.V(3).Infof("[BenchMark] Result for %s---------------\n", time.Now().UTC().UTC().String())
	glog.V(3).Infof("[BenchMark]:goroutine count: [%d], Total duration is %d mins\n", goroutionNum, totalDuration)
	return nil
}
```

### generator function
```
// here you can write your code to generate items
func Producer(ch chan string) error {
	for i:=0;i<10000;i++{
	    ch <- fmt.Sprintf("item #%d",i )
		}
	}
	close(ch)
	return nil
}
```

### consumer

```
func Consumer(id int, ch chan string, done chan bool) {
	for {
		value, ok := <-ch
		if ok {
			// Simulation process
			fmt.Printf("I am work #%d, I am handling %s",id, value)
		} else {
			break
		}
	}
	done <- true
}

```

OK, thanks for your view. 
