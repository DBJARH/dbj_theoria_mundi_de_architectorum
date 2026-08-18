
## Conceptual view: Why Harness

![alt text](call-harness-model.png)

**To emulate normal conversation with Model. From Callers point of View**

## Logical View: What is the harness

### Why the Book

By using the "Book" Concept, Model receives the full conversation context with each request.

![alt text](call-harness-model-books.png)

Book = Full Calling Site Context + Original Prompt

### Harness Logic

![alt text](calling_site_harnes_logic.png)

Harness "duties"

1. Communication to/from the Caller
2. Communication fo/from the Model
3. Actuators communication to tools available in the calling site and other harnesses available in the calling site
4. Collection the context from the calling site

## Physical View: Harness and the Runtime Environment


Page is useful abstraction in this protocol.

1. class Page
   1. subclass PageReply
      1. subclass PageErrorReply

That is logical, will be implemented as JSON. To travel with textual reply.


## Application View

```py
book = initial_book()
for turn in range(MAX_TURNS):
    page = model.generate(book)
    if page.is_final():
        reply_to_caller(page.answer)
        break
    tool_results = execute_tools(page.tool_calls)
    book = book.append(tool_results)
```


