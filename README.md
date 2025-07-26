# Basic-REST-API-with-Flask
This project creates a simple API for managing a collection of books. It uses Flask to handle HTTP requests and a dictionary to store the book data in memory (for simplicity, a real-world application would use a database).
from flask import Flask, jsonify, request

app = Flask(__name__)

# In-memory storage for books (acts as a simple database)
books = [
    {'id': 1, 'title': 'The Lord of the Rings', 'author': 'J.R.R. Tolkien'},
    {'id': 2, 'title': 'Pride and Prejudice', 'author': 'Jane Austen'}
]

# A simple counter for new book IDs
next_book_id = 3

@app.route('/books', methods=['GET'])
def get_books():
    """Returns a list of all books."""
    return jsonify({'books': books})

@app.route('/books/<int:book_id>', methods=['GET'])
def get_book(book_id):
    """Returns a single book by its ID."""
    book = next((book for book in books if book['id'] == book_id), None)
    if book:
        return jsonify({'book': book})
    return jsonify({'error': 'Book not found'}), 404

@app.route('/books', methods=['POST'])
def add_book():
    """Adds a new book to the collection."""
    global next_book_id
    if not request.json or 'title' not in request.json or 'author' not in request.json:
        return jsonify({'error': 'Invalid data'}), 400
    
    new_book = {
        'id': next_book_id,
        'title': request.json['title'],
        'author': request.json['author']
    }
    books.append(new_book)
    next_book_id += 1
    return jsonify({'message': 'Book added successfully', 'book': new_book}), 201

@app.route('/books/<int:book_id>', methods=['PUT'])
def update_book(book_id):
    """Updates an existing book by its ID."""
    book = next((book for book in books if book['id'] == book_id), None)
    if not book:
        return jsonify({'error': 'Book not found'}), 404

    if not request.json:
        return jsonify({'error': 'Invalid data'}), 400
    
    book['title'] = request.json.get('title', book['title'])
    book['author'] = request.json.get('author', book['author'])
    
    return jsonify({'message': 'Book updated successfully', 'book': book})

@app.route('/books/<int:book_id>', methods=['DELETE'])
def delete_book(book_id):
    """Deletes a book by its ID."""
    global books
    initial_len = len(books)
    books = [book for book in books if book['id'] != book_id]
    
    if len(books) < initial_len:
        return jsonify({'message': 'Book deleted successfully'}), 200
    
    return jsonify({'error': 'Book not found'}), 404

if __name__ == '__main__':
    app.run(debug=True)
