# Konkurensi
Konkurensi adalah komposisi / struktur dari berbagai proses yang berjalan secara bersamaan. Fitur untuk melakukan konkurensi dalam golang adalah Go Routine.

## Go Routine
- Sebuah thread yang ringan, hanya dibutuhkan 2kB memori untuk menjalankan sebuah go routine
- Aksi go routine bersifat asynchronous, jadi tidak saling menunggu dengan go routine yang lain.
- Proses yang hendak dieksekusi sebagai go routine harus berupa fungsi tanpa return yang dipanggil dengan kata kunci go

```
package main

func Salam(s string) {
    for i := 0; i <= 10; i++ {
		println(s)
		time.Sleep(1000 * time.Millisecond)
	}
}

func main() {
    go Salam("Selamat Pagi")
    Salam("Selamat Malam")
}
```
- Go routine jalan di multi core processor, dan bisa diset mau jalan di berapa core.
```
package main

func Salam(s string) {
    for i := 0; i <= 10; i++ {
		println(s)
		time.Sleep(1000 * time.Millisecond)
	}
}

func main() {
    runtime.GOMAXPROCS(1)

    go Salam("Selamat Pagi")
    Salam("Selamat Malam")
}
```

## Channel
- Untuk mengsinkronkan satu go routine dengan go routine lainnya, diperlukan channel
- Channel digunakan untuk menerima dan mengirim data antar go routine.
- Channel bersifat blocking / synchronous. Pengiriman dan penerimaan ditahan sampai sisi yang lain siap.
- Channel harus dibuat sebelum digunakan, dengan kombinasi kata kunci make dan chan
- Aliran untuk menerima / mengirim data ditunjukkan dengan arah panah

```
package main

import "runtime"

func main() {
	var pesan = make(chan string)
	println("kirim data", "Jacky")
	pesan <- "Jacky"

    // akan error karena tidak ada go routine lain yang menangkap channel
	println("terima data", <-pesan)	
}
```

```
package main

func main() {
	var pesan = make(chan string)

	println("kirim data", "Jacky")
	pesan <- "Jacky"

    go func() {
		println("terima data", <-pesan)
	}()

	
}
```

```
package main

func main() {
	var pesan = make(chan string)

	go func() {
		println("terima data", <-pesan)
	}()

	println("kirim data", "Jacky")
	pesan <- "Jacky"

}
```

```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}
    
    var pesan = make(chan string)
	
    // error karena go routine yang melakukan penerimaan data hanya sekali, sementara pengiriman dilakukan 4 kali 
	go func() {
		println("terima data", <-pesan)
	}()

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
}

```

```
package main

func main() {
    a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

    var pesan = make(chan string)
	
    go func() {
        for {
		    println("terima data", <-pesan)
        }
	}()

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
}
```

## Channel dengan buffer
- Panjang buffer ditambahkan pada fungsi make sebagai argumen kedua
- Buffering menyebabkan pengiriman dan penerimaan data berlangsung secara asynchronous
- Pengiriman ke kanal buffer akan ditahan bila buffer telah penuh. Penerimaan akan ditahan saat buffer kosong.
- Jika pengiriman data melebihi panjang buffer, maka akan diperlakukan secara synchronous.
```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

    var pesan = make(chan string, 3)
	
	go func() {
		for {
			println("terima data", <-pesan)
		}
	}()

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
}
```  

```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

	var pesan = make(chan string, len(a))

	go func() {
		println("terima data", <-pesan)
	}()

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
}

```

```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}
	var pesan = make(chan string, len(a))
	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
}
```

## Range dan Close
- Range merupakan perulangan dari sebuah channel
```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

	var pesan = make(chan string, len(a)-1)

	go func() {
		for i := range pesan {
			println("terima data", i)
		}
	}()

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}

}
```
- Pengirim bisa menutup sebuah channel untuk menandai sudah tidak ada data yang dikirim lagi.
- Penutupan ini hanya optional. Artinya pengirim boleh melakukan close maupun tidak.
- Yang melakukan close hanya pengirim. Karena jika yang melakukan close adalah penerima, dan ada routine yang melakukan pengrimana akan menyebabkan panic.
- Penerima bisa menambahkan pengecekan, jika masih ada data yang dikirim maka akan diterima. 
```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

	var pesan = make(chan string, len(a)-1)

	go func() {
		for {
			println("terima data", <-pesan)
			close(pesan)
		}
	}()

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
}

```

```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

	var pesan = make(chan string, len(a)-1)

	go func() {
		for {
			println("terima data", <-pesan)
		}
	}()

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
	close(pesan)
}
```

```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

	var pesan = make(chan string, len(a))

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
	close(pesan)
}

```

```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

	var pesan = make(chan string, len(a))

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
	close(pesan)

	for {
		println("terima data", <-pesan)
	}
}
```

```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

	var pesan = make(chan string, len(a))

	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
	close(pesan)

	for {
        // dilakukan pengecekan agar tidak looping forefer
		if v, ok := <-pesan; ok {
			println("terima data", v)
		} else {
			break
		}
	}
}

```

```
package main

func main() {
	a := []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"}

	var pesan = make(chan string, len(a))

    // menggunakan range jauh lebih simple
	for _, s := range a {
		println("kirim data", s)
		pesan <- s
	}
	close(pesan)

	for i := range pesan {
		println("terima data", i)
	}
}
```

## Select
- Channel diperlukan untuk pertukaran data antar go routine
- Jika melibatkan lebih dari satu go routine, diperlukan fungsi kontrol melalui select
- Select akan menerima secara acak mana data yang terlebih dahulu tersedia
```
package main

func main() {
	var pesan = make(chan string)
	var c = make(chan int)

	go func() {
		for _, s := range []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"} {
			pesan <- s
		}
	}()

	go func() {
		c <- 5
	}()

	select {
	case i := <-c:
		println("terima data", i)
	case s := <-pesan:
		println("terima data", s)
	}
}
```

```
package main

func main() {
	var pesan = make(chan string)
	var c = make(chan int)

	go func() {
		for _, s := range []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"} {
			pesan <- s
		}
	}()

	go func() {
		c <- 5
	}()

	for a := 0; a <= 4; a++ {
		select {
		case i := <-c:
			println("terima data", i)
		case s := <-pesan:
			println("terima data", s)
		}
	}
}
```

```
package main

func main() {
	var pesan = make(chan string)
	var c = make(chan int)

	go func() {
		for _, s := range []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"} {
			pesan <- s
		}
	}()

	go func() {
		c <- 5
	}()

	for a := 0; a <= 5; a++ {
		select {
		case i := <-c:
			println("terima data", i)
		case s := <-pesan:
			println("terima data", s)
		}
	}
}

```
## Select Default
- Jika saat select tidak ada channel yang siap diterima maka akan dijalankan baris kode default
```
package main

func main() {
	var pesan = make(chan string)
	var c = make(chan int)

	go func() {
		for _, s := range []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"} {
			pesan <- s
		}
	}()

	go func() {
		c <- 5
	}()

	for a := 0; a <= 5; a++ {
		select {
		case i := <-c:
			println("terima data", i)
		case s := <-pesan:
			println("terima data", s)
        default :
            println("tidak ada penerimaan data")
		}
	}
}

``` 

## Select Timeout
- Teknik tambahan untuk mengakhiri select jika tidak ada penerimaan data
```
package main

import (
	"fmt"
	"time"
)

func main() {
	var pesan = make(chan string)
	var c = make(chan int)

	go func() {
		for _, s := range []string{"Jacky", "Jet Lee", "Bruce Lee", "Samo Hung"} {
			pesan <- s
		}
	}()

	go func() {
		c <- 5
	}()

loop:
	for {
		select {
		case i := <-c:
			println("terima data", i)
		case s := <-pesan:
			println("terima data", s)
		case <-time.After(time.Second * 5):
			fmt.Println("timeout. tidak ada aktivitas selama 5 detik")
			break loop
		}
	}
}

```

## Sync Mutex
- Channel dipakai untuk komunikasi antar go routine
- Jika tidak ingin berkomunikasi karena ngin memastikan hanya satu goroutine yang dapat mengakses suatu variabel pada satu waktu untuk menghindari konflik, digunakan sync mutex
- mutex adalah mutual exclusion dengan fungsi `Lock` dan `Unlock`
```
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter aman digunakan secara konkuren.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc meningkatkan nilai dari key.
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock sehingga hanya satu goroutine pada satu waktu yang dapat
	// mengakses map c.v.
	c.v[key]++
	c.mux.Unlock()
}

// Value mengembalikan nilai dari key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock sehingga hanya satu gorouting pada satu waktu yang dapat
	// mengakses map c.v.
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("key")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("key"))
}

``` 