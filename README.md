#to do
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "github.com/gorilla/mux"
    "strconv"
)


type Todo struct {
    ID     int    `json:"id"`
    Title  string `json:"title"`
    Status string `json:"status"`
}

var todos []Todo
var nextID = 1

func main() {
	router := mux.newRouter()

	router.HandleFunc("/todos", getTodos).Methods("GET")
	router.HandleFunc("/todos/{id}", getTodo).Methods("GET")
	router.HandleFunc("/todos", createTodo).Methods("POST")
	router.HandleFunc("/todos/{id}", updateTodo).Methods("PUT")
	router.HandleFunc("/todos/{id}", deleteTodo).Methods("DELETE")

	log.Println("server is running on port 8080...")
	log.Fatal(http.ListenAndServe(":8080", router))

}

func getTodos(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("content-Type", "application/json")
	json.NewEncoder(w).Encode(todos)
}


func getTodo(w http.ResponseWriter, r *http.Request){
	w.Header().Set("content-Type", "application/json")
	params := mux.vars(r)
	id, err:= strconv.Atoi(params["id"])
	if err != nil {
		http.Error(w, "Invalid ID", http.StatusBadRequest)
		return
	}

	for _, todo := range todos {
		if todo.ID == id {
			json.NewEncoder(w).Encode(todo)
			return
		}
	}
	http.Error(w, "Todo Not Found", http.StatusNotFound)

	func createTodo(w http.ResponseWriter, r *http.Request) {
		var newTodo Todo
	
		// Handle potential error during decoding
		err := json.NewDecoder(r.Body).Decode(&newTodo)
		if err != nil {
			http.Error(w, "Invalid request payload", http.StatusBadRequest)
			return
		}
	
		newTodo.ID = nextID
		nextID++
		todos = append(todos, newTodo)
	
		w.Header().Set("Content-Type", "application/json")
		json.NewEncoder(w).Encode(newTodo)
	}
	

func updateTodo(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    id, err := strconv.Atoi(params["id"])
    if err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }

	var updatedTodo Todo
    _ = json.NewDecoder(r.Body).Decode(&updatedTodo)

	for i, todo := range todos {
        if todo.ID == id {
            todos[i].Title = updatedTodo.Title
            todos[i].Status = updatedTodo.Status
            json.NewEncoder(w).Encode(todos[i])
            return
        }
    }
    http.Error(w, "Todo not found", http.StatusNotFound)
}


func deleteTodo(w http.ResponseWriter, r *http.Request) {
    params := mux.Vars(r)
    id, err := strconv.Atoi(params["id"])
    if err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }


	for i, todo := range todos {
        if todo.ID == id {
            todos = append(todos[:i], todos[i+1:]...) // Remove the todo from the slice
            w.WriteHeader(http.StatusNoContent)       // Return 204 No Content
            return
        }
    }
    http.Error(w, "Todo not found", http.StatusNotFound)
}
}








