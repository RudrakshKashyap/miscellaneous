An API, or Application Programming Interface, is a set of rules and protocols that define how different software applications or components can interact and communicate with each other. 


# REpresentational State Transfer (REST)
An architectural style for designing networked applications(not a strict protocol or standard), with principles like statelessness, uniform interface, and resource orientation.


### **Key Principles (Constraints) of REST**

For an API to be truly "RESTful," it should adhere to these six constraints:

1.  **Client-Server:** Separation of concerns. The client handles the UI/user experience, the server handles data storage and logic. They evolve independently.
2.  **Stateless:** Each request from the client to the server must contain all the information needed to understand and process the request. The server does not store any client context between requests.
3.  **Cacheable:** Responses must define themselves as cacheable or not. This improves performance and scalability on the client-side.
4.  **Uniform Interface:** The interface between client and server is standardized. This is achieved through:
    *   **Resource Identification in Requests:** Resources are identified in requests (e.g., using URLs).
    *   **Resource Manipulation through Representations:** Clients manipulate resources through representations (e.g., JSON) which contain enough information to do so.
    *   **Self-descriptive Messages:** Each message includes enough information to describe how to process it.
    *   **HATEOAS (Hypermedia as the Engine of Application State):** Responses should include hyperlinks to related resources, guiding the client on what actions can be taken next.
5.  **Layered System:** The architecture can be composed of multiple layers (e.g., security, load-balancing, application logic). The client cannot tell if it's connected directly to the end server or to an intermediary.
6.  **Code on Demand (Optional):** Servers can temporarily extend or customize client functionality by transferring executable code (e.g., JavaScript). This is the least used constraint.
---

### **Best Practices**

*   **Use Nouns, Not Verbs:** The endpoint should be a noun (resource), not a verb. `/users` is good, `/getUsers` is bad.
*   **Use Plural Nouns:** Use plural nouns for consistency (e.g., `/books`, `/users`).
*   **Use HTTP Status Codes Correctly:** Don't just return `200 OK` for every request. Use the correct code for the situation.
*   **Version Your API:** Include the version number in the URL (`/api/v1/books`) or in the request header. This prevents breaking changes for existing clients.
*   **Provide Filtering, Sorting, and Pagination:** For collections, allow parameters like `?limit=10&offset=20&sort=title&author=Smith`.
*   **Use SSL/TLS (HTTPS):** Always. For security.
*   **Good Documentation:** Use tools like **OpenAPI (Swagger)** to create clear, interactive documentation for consumers of your API.

---
<br />
<br />
<br />

# soap - todo

# [GraphQL](https://www.youtube.com/watch?v=1VpqNYbdB6k) - No overfetching, No underfetching
- GraphQL is a data query and manipulation language that allows specifying what data is to be retrieved ("declarative data fetching") or modified. 
- A GraphQL server can process a client query using data from separate sources and present the results in a unified graph.
- The language is not tied to any specific database or storage engine. There are several open-source runtime engines for GraphQL.

| **Aspect**                             | **Why REST is Better**                                                | **Notes**                                                                               |
| -------------------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Caching**                            | REST uses standard HTTP caching (ETag, Last-Modified, Cache-Control). | GraphQL queries are usually POST requests, so caching requires extra setup.             |
| **Simplicity**                         | REST is easier to understand and implement.                           | Uses standard HTTP verbs and URLs; no new query language needed.                        |
| **Predictability**                     | Endpoints and responses are fixed and predictable.                    | Easier to secure and maintain, especially for large teams.                              |
| **Performance for Simple CRUD**        | Minimal overhead; straightforward request/response.                   | GraphQL may be overkill for simple endpoints.                                           |
| **Error Handling**                     | Uses standard HTTP status codes.                                      | GraphQL wraps errors in a 200 response, which can be confusing.                         |
| **Security**                           | Easier to control which endpoints are exposed.                        | GraphQL’s flexible queries require additional protections (query depth, cost analysis). |
| **Incremental Adoption**               | Easy to add or version new endpoints.                                 | GraphQL often requires more planning, especially for large existing APIs.               |
| **Streaming / Long-lived connections** | REST + HTTP/2 or SSE is straightforward.                              | GraphQL subscriptions exist but need a subscription server to manage connections and push events.                            |

---


# [gRPC](https://www.youtube.com/watch?v=gnchfOojMk4)

![](https://miro.medium.com/v2/resize:fit:1400/1*ieImkZ0Dgv4GM4j4mrqcFw.png)
RPC is a protocol that allows a computer program to cause a procedure to execute in a different address space (commonly on another computer), without the programmer explicitly coding the details for this remote interaction. The goal is to make a remote call look and feel as much like a local function call as possible.

**Key Architectural Components:**
The RPC architecture involves several key components that work together to abstract the network communication:
*   **Client & Server:** The client makes the call, and the server hosts and executes the procedure.
*   **Client Stub:** A proxy on the client side. It marshals (serializes) the procedure arguments into a network message and sends the request.
*   **Server Stub:** Receives the request, unmarshals (deserializes) the arguments, calls the actual server procedure, and then marshals the result to send back.
*   **RPC Runtime:** The underlying protocol that handles message transmission, retries, and acknowledgment.

## MY UNDERSTANDING

So basically, all gRPC is doing is that it abstracts away the inner implementation of HTTP communication, making life easier for developers. Also the transfered data is in Protobuf format, which is compressed and hence faster, i.e. more data transfer in one packet. If we are developing an Software(eg- discord windows app), we are going to need the HTTP libraries in code to send packets, and updating/maintaing those libraries is an hussle also error handeling. But with gRPC we dont need to worry about that, google is maintaing and updating those libraries. We dont need to do all this in a browser though, since browser has all the Http libraries and stuff. Btw gRPC doesn't work on browsers(need `gRPC-Web` for that), since it strictly rely on HTTP/2(can use multiple streams for multiple function calls simultaneously, which would not be possible in rest + it is bidirectional) and sends packets in Protobuf format not in simple json/text format. gRPC is best for internal microservice communication. Also it has Strong typing across network boundaries.

```python
# What you DON'T have to do with gRPC:
import http.client
import json

headers = {'Content-Type': 'application/json'}
body = json.dumps({'user_id': '123', 'include_profile': True})
connection = http.client.HTTPSConnection("api.example.com")
connection.request("POST", "/v1/users", body, headers)
response = connection.getresponse()
data = json.loads(response.read())
# Handle status codes, retries, timeouts, connection pooling...

# What you DO with gRPC:
response = stub.GetUser(UserRequest(user_id="123"))
```

```pgsql
+--------------------+------------------------+-----------------------+
| Feature            | REST                   | gRPC                  |
+--------------------+------------------------+-----------------------+
| Transport          | HTTP/1.1 or HTTP/2     | HTTP/2 (full streams) |
| Data Format        | JSON / text            | Protobuf (binary)     |
| Typing / Contract  | Weak (documentation)   | Strong (.proto enforced) |
| Streaming          | Request/Response only  | Unary, Server, Client, Bi-directional |
| Multiplexing       | No (1 request per conn)| Yes (multiple RPCs on single conn) |
| Browser Support    | Native (XHR/fetch)     | Limited (needs gRPC-Web) |
| Real-time usage    | Needs WebSockets/SSE   | Native streaming      |
| Best Use Case      | Public APIs, simple apps| Internal microservices, low-latency/high throughput |
+--------------------+------------------------+-----------------------+

        REST                   gRPC                     gRPC-Web
   (HTTP/1.1 or HTTP/2)    (HTTP/2 native)           (HTTP/1.1/2 via fetch/XHR)
        JSON                  Protobuf                 Protobuf (wrapped)
   Request/Response      Unary + Streaming         Unary + Server-streaming
       Weak typing           Strong typing             Strong typing
   Browser native          Browser limited           Browser compatible
```

### 🚀 What is gRPC?

gRPC is a high-performance, open-source universal RPC framework that was initially developed by Google. It is a specific implementation of the RPC model with modern optimizations.

**Core Concepts of gRPC:**
*   **Service Definition:** Services are defined in a `.proto` file using Protocol Buffers. The service definition specifies the methods that can be called remotely along with their parameters and return types.
*   **Protocol Buffers (Protobuf):** This is the **Interface Definition Language (IDL) and message format** for gRPC. You define your data structures and services in a `.proto` file, and the Protobuf compiler (`protoc`) generates client and server code in your target language.
*   **HTTP/2 as Transport:** gRPC uses HTTP/2 as its underlying protocol, which provides benefits like multiplexing (multiple streams over a single TCP connection), header compression, and bidirectional communication.

**gRPC Communication Models:**
gRPC supports four types of service methods, offering great flexibility:
1.  **Unary RPC:** The client sends a single request and the server returns a single response (like a normal function call).
2.  **Server Streaming RPC:** The client sends a single request and the server returns a stream of messages.
3.  **Client Streaming RPC:** The client sends a stream of messages to the server, which then returns a single response.
4.  **Bidirectional Streaming RPC:** Both client and server send streams of messages to each other independently.


## **Internal Working Flow (Step-by-Step)**

> The stub is the auto-generated code created by the Protocol Buffers compiler (protoc) from your .proto file. It acts as a client-side proxy that makes remote calls appear like local method calls. Significantly speeding up the development.


**For a unary RPC call:**

1. **Client creates a request object** (e.g., `UserRequest`).
2. **Client stub serializes request** using protobuf.
3. **Request sent over HTTP/2** to server.
4. **HTTP/2 demultiplexes stream** on server side.
5. **Server deserializes request** into an object.
6. **Server executes service method**, generates response object.
7. **Server serializes response** using protobuf.
8. **Response sent back over HTTP/2** stream to client.
9. **Client stub deserializes response**.
10. **Client receives the result** as if it was a local method call.

> For streaming RPCs, steps 2–9 repeat per message in the stream.
