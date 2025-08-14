# Assignment

## Brief

Write the Python codes for the following questions.

## Instructions

Paste the answer as Python in the answer code section below each question.

### Question 1

Question: Implement a simple Thrift server and client that defines a `Student` struct with fields `name` (string), `age` (integer), and `courses` (list of strings). Include a service `School` with a method `enrollCourse` that takes a `Student` record and a course name, adds the course to the student's course list, and returns the updated `Student` record.

Answer:

```python
%%writefile ../schema/student.thrift

struct Student {
  1: required string name,
  2: required i32 age,
  3: optional list<string> courses
}

service School {
  Student enrollCourse(1: required Student student, 2: required string course)

Overwriting ../schema/student.thrift

%%writefile ../student_server.py
import thriftpy2
from thriftpy2.rpc import make_server

student_thrift = thriftpy2.load("student.thrift", module_name="student_thrift")


# Define the service implementation
class SchoolService:
    def enrollCourse(self, student, course):
        if student.courses is None:
            student.courses = []
        student.courses.append(course)
        return student

# Start the server
server = make_server(student_thrift.School, SchoolService(), host='localhost', port=6000)
print("🎓 School server is running on port 6000...")
server.serve()

Writing ../student_server.py

import thriftpy2
from thriftpy2.rpc import make_client

student_thrift = thriftpy2.load("schema/student.thrift", module_name="student_thrift")

try:
    school = make_client(student_thrift.School, 'localhost', 6000)
    print("✅ Connected to School server")

    alice = student_thrift.Student(name="Alice", age=20, courses=["math", "history"])
    print("📚 Before enrollment:", alice.courses)

    updated = school.enrollCourse(alice, "physics")
    print("🎓 After enrollment:", updated.courses)

except Exception as e:
    print("❌ Error:", e)

✅ Connected to School server
📚 Before enrollment: ['math', 'history']
🎓 After enrollment: ['math', 'history', 'physics']

```

### Question 2

Question: Implement a simple Protocol Buffers server and client that defines a `Book` message with fields `title` (string), `author` (string), and `page_count` (integer). Include a service `Library` with a method `checkoutBook` that takes a `Book` message and returns the same `Book` message.

Answer:

```python
# Protobuf schema (book.proto)
syntax = "proto3";

package library;

message Book {
  string title = 1;
  string author = 2;
  int32 page_count = 3;
}

service Library {
  rpc CheckoutBook(Book) returns (Book);
}

# Protobuf server (book_server.py)
import grpc
from concurrent import futures
import time

import book_pb2
import book_pb2_grpc

class LibraryServicer(book_pb2_grpc.LibraryServicer):
    def CheckoutBook(self, request, context):
        print(f"📚 Checkout request: {request.title} by {request.author}")
        return request  # Just echoing back the same Book

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    book_pb2_grpc.add_LibraryServicer_to_server(LibraryServicer(), server)
    server.add_insecure_port('[::]:50051')
    print("📘 Library server running on port 50051...")
    server.start()
    try:
        while True:
            time.sleep(86400)
    except KeyboardInterrupt:
        server.stop(0)

if __name__ == '__main__':
    serve()


# Protobuf client (book_client.py)
import grpc
import book_pb2
import book_pb2_grpc

def run():
    channel = grpc.insecure_channel('localhost:50051')
    stub = book_pb2_grpc.LibraryStub(channel)

    book = book_pb2.Book(title="1984", author="George Orwell", page_count=328)
    print("📖 Sending book to checkout:", book.title)

    response = stub.CheckoutBook(book)
    print("✅ Received book back:", response.title, "by", response.author)

if __name__ == '__main__':
    run()

📘 Library server running on port 50051...
📖 Sending book to checkout: 1984
📚 Checkout request: 1984 by George Orwell
✅ Received book back: 1984 by George Orwell

```

## Submission

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.
