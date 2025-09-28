<h1>Apuntes de Go</h1>

***Index***:
<!-- TOC -->
  * [Comandos útiles](#comandos-útiles)
    * [Correr un archivo `.go`](#correr-un-archivo-go)
    * [Compilar/buildear un archivo `.go` y luego correr el archivo compilado](#compilarbuildear-un-archivo-go-y-luego-correr-el-archivo-compilado)
    * [Inicializar manejador de módulos](#inicializar-manejador-de-módulos)
    * [Limpiar y organizar las dependencias](#limpiar-y-organizar-las-dependencias)
    * [Obtener dependencia externa (que no es parte de la *standard library* de Go)](#obtener-dependencia-externa-que-no-es-parte-de-la-standard-library-de-go)
    * [Testing](#testing)
  * [Algunos ejemplos](#algunos-ejemplos)
  * [Links útiles](#links-útiles)
<!-- TOC -->

---

## Comandos útiles

### Correr un archivo `.go`

```bash
    go run example.go
```

### Compilar/buildear un archivo `.go` y luego correr el archivo compilado

```bash
    go build example.go
    
    ./example
```

### Inicializar manejador de módulos

```bash
    go mod init example
```

### Limpiar y organizar las dependencias

```bash
    go mod tidy
```

### Obtener dependencia externa (que no es parte de la *standard library* de Go)

```bash
    go get rsc.io/quote
```

### Testing

```bash
    # Correr los tests (funciones del tipo 'TestSomeFunction(t *testing.T){...}' en los archivos {FILE-NAME}_test.go)
    go test
    
    # Correr los tests con coverage
    go test -cover
    
    # Generar un archivo con el coverage y luego visualizarlo en un navegador
    go test -coverprofile=coverange.out
    go tool cover -html=coverange.out
    
    # Generar un archivo de profiling y luego analizarlo
    go test -cpuprofile=cpu.out
    go tool pprof cpu.out # Entra en un modo interactivo que permite ingresar por ejemplo "help" para ver otros comandos
```

## Algunos ejemplos

```go
    package main
    
    import (
        "fmt"
        "math"
        "strconv"
    
        "rsc.io/quote"
    )
    
    func main() {
        // Conversión de tipos
        var integer16 int16 = 50
        var integer32 int32 = 100
        fmt.Println(int32(integer16) + integer32) // 150
    
        numberString := "100"
        i, _ := strconv.Atoi(numberString)
        fmt.Println(i + i) // 200
    
        n := 42
        s = strconv.Itoa(n)
        fmt.Println(s + s) // 4242
    
        // Formatear un string
        fullName := "Javi Tapia \t(alias \"javito\")\n"
        fmt.Println(fullName) // Javi Tapia      (alias "javito")
        firstName, lastName, age := "Javi", "Tapia", 40
        fmt.Printf("Hola, soy %s y tengo %d años\n", firstName, age) // Hola, soy Javi y tengo 40 años
        greeting := fmt.Sprintf("Hola, soy %s y tengo %d años", firstName, age)
        fmt.Println(greeting) // Hola, soy Javi y tengo 40 años
    
        // Solicitar datos al usuario
        fmt.Print("Ingrese su nombre: ")
        fmt.Scanln(&firstName)
        fmt.Print("Ingrese su edad: ")
        fmt.Scanln(&age)
        fmt.Printf("Hola, soy %s y tengo %d años\n", firstName, age) // Hola, soy Javi y tengo 40 años
        
        // Slice
        weekDays := []string{"Domingo", "Lunes", "Martes", "Miercoles", "Jueves", "Viernes", "Sabado"}
        fmt.Println(weekDays)
        sliceWeekDays := weekDays[0:5]
        fmt.Println(sliceWeekDays)
        fmt.Println(cap(sliceWeekDays)) // 7 -> Capacidad del slice original, no de lo que se está tomando (0:5)
        sliceWeekDays = append(sliceWeekDays, "Osvaldo")
        fmt.Println(sliceWeekDays) // [Domingo Lunes Martes Miercoles Jueves Osvaldo]
        sliceTwo := []string{"Urano", "Plutón"}
        sliceWeekDays = append(sliceWeekDays, sliceTwo...)
        fmt.Println(sliceWeekDays) // [Domingo Lunes Martes Miercoles Jueves Osvaldo Urano Plutón]
        sliceWeekDaysDeleteJueves := append(sliceWeekDays[:4], sliceWeekDays[5:]...)
        fmt.Println(sliceWeekDaysDeleteJueves) // [Domingo Lunes Martes Miercoles Osvaldo Urano Plutón]
        
        // Punteros
        var x int = 10
        var p *int = &x
        fmt.Println(x) // 10
        fmt.Println(p) // 0x1400010e4f0
        editar(p)
        fmt.Println(x) // 20
        person := Person{"Javi", 40, "javi@gmail.com"}
        person.sayHello()
    }
    
    func editar(x *int) {
        *x = 20
    }
    
    func (p *Person) sayHello() {
        fmt.Println("Hi, my name is", p.name) // Hi, my name is Javi
    }
    
    type Person struct {
        name  string
        age   int
        email string
    }
```

## Links útiles
- https://go.dev/
- https://gobyexample.com/
- https://github.com/alexroel/curso-golang (curso Go Udemy)
- https://github.com/joho/godotenv
- https://github.com/air-verse/air
- https://github.com/gorilla/mux
- https://github.com/go-yaml/yaml
- https://gorm.io/index.html
