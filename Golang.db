package main

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"net/http"

	_ "github.com/go-sql-driver/mysql"
)

var db *sql.DB

func initDB() {
	var err error
	db, err = sql.Open("mysql", "root:1234@tcp(localhost:3306)/usersdb")
	if err != nil {
		fmt.Println("Erro ao abrir o banco de dados:", err)
		return
	}

	err = db.Ping()
	if err != nil {
		fmt.Println("Erro ao conectar ao banco de dados:", err)
		return
	}

	fmt.Println("Conectado ao banco de dados!")
}

type User struct {
	ID   string `json:"id"`
	Name string `json:"name"`
	Age  uint   `json:"age"`
}

func getUser(w http.ResponseWriter, r *http.Request) {
	rows, err := db.Query("SELECT * FROM users")
	if err != nil {
		http.Error(w, "Erro ao consultar o banco de dados", http.StatusInternalServerError)
		return
	}
	defer rows.Close()

	var users []User

	for rows.Next() {
		var user User
		err := rows.Scan(&user.ID, &user.Name, &user.Age)
		if err != nil {
			http.Error(w, "Erro ao escanear as linhas", http.StatusInternalServerError)
			return
		}
		users = append(users, user)
	}

	data, err := json.Marshal(users)
	if err != nil {
		http.Error(w, "Erro com os usuários", http.StatusInternalServerError)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	w.Write(data)
}

func createUser(w http.ResponseWriter, r *http.Request) {
	var newUser User
	decoder := json.NewDecoder(r.Body)
	err := decoder.Decode(&newUser)
	if err != nil {
		http.Error(w, "Erro na decodificação", http.StatusBadRequest)
		return
	}

	_, err = db.Exec("INSERT INTO users (id, name, age) VALUES (?, ?, ?)", newUser.ID, newUser.Name, newUser.Age)
	if err != nil {
		http.Error(w, "Erro ao inserir no banco de dados", http.StatusInternalServerError)
		return
	}

	fmt.Println("Novo usuário criado: ", newUser)
	w.WriteHeader(http.StatusCreated)
}

func main() {
	initDB()

	http.HandleFunc("/", getUser)
	http.HandleFunc("/create", createUser)
	http.ListenAndServe(":8000", nil)
}
