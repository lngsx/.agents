
This is not exactly an `AGENTS.md`, it's a central repository for me to store all the agent behaviors I often use. Treat this as a raw text bank, cherry-pick and hand-copy whatever I need for a new project/agent.

---

## Ruby

### Formatting & Structure

- **Indentation**: 2 spaces (no tabs)
- **Blank Lines**: One between method definitions, two between classes
- **Method Organization**: Public first, private/protected at bottom
- **Method Visibility**: Explicit `private` / `protected` keywords

### Blank Line Before Final Statement

Always add a blank line before the final statement in a method. This visually separates the conclusion from the setup.

```ruby
def index
  @documents = Document.all

  render json: @documents
end

def process(id, label)
  record = Record.find(id)
  result = record.process(label)

  render json: result
end
```

### Functional Style

Prefer functional, immutable Ruby where it doesn't hurt readability. Favour `map`, `filter_map`, `reduce`, `then`, `yield_self`. Avoid mutating in place when a transformation reads more clearly.

```ruby
# Good
ids = params[:ids]
names = User.where(id: ids).map(&:name)
slugs = names.map { |n| n.parameterize }

# Avoid when functional reads better
results = []
User.where(id: ids).each { |u| results << u.name.parameterize }
```

Use `unless` for simple negation guards instead of `if !`.

### Modern Ruby

Prefer modern Ruby features where they improve clarity:

- Pattern matching (`case/in`) for destructuring complex data
- Numbered block parameters (`it`) for trivial one-liners
- Hash shorthand (`{ name:, email: }`) when variable names match keys
- Endless methods (`def to_s = "#{first} #{last}"`) for trivial one-liners

### Variable Extraction

When a nested or chained expression gets complex, pull it into a named variable. Clarity beats brevity.

```ruby
# Good
attachment_url = document.file.blob.service_url(expires_in: 1.hour)
metadata = { url: attachment_url, size: document.file.blob.byte_size }

render json: metadata

# Avoid
render json: { url: document.file.blob.service_url(expires_in: 1.hour), size: document.file.blob.byte_size }
```

### Grouped Variable Assignment

Group variables by concern. Separate groups with a blank line.

```ruby
def query_result(record_ids, started_at, ended_at, label, incoming_email)
  @records = SlipSummary.where(id: record_ids)
  @total_amount = @records.total_amount

  @range = QueryRange.new(started_at:, ended_at:, label:)
  @label = label

  @incoming_email = incoming_email
  @emoji = random_emoji

  mail(
    to: EmailAddress.all.pluck(:address),
    subject: "All in #{@label} (#{@records.count} items) #{@emoji}"
  )
end
```

### Structs — Use dry-struct

When a typed value object is needed, use `dry-struct`. Avoid plain `Struct` or `OpenStruct`.

```ruby
module Types
  include Dry.Types()
end

class QueryRange < Dry::Struct
  attribute :started_at, Types::JSON::DateTime
  attribute :ended_at, Types::JSON::DateTime
  attribute :label, Types::Strict::String
end
```



### RBS Inline Type Annotations

Use `# @rbs` inline annotations for method signatures. This project uses ruby-lsp, which understands RBS inline comments. It is the preferred documentation standard — better than YARD (which is Ruby-specific and not machine-checkable) and better than annotate (which is schema-level only). You do not need a full RBS/Steep setup for these comments to be useful; ruby-lsp will surface them, and they serve as a clear, modern contract.

Format:

```ruby
# @rbs (id: Integer, label: String) -> Document
def find_document(id, label)
  Document.find_by!(id:, label:)
end
```

Add inline annotations to:
- Public model methods
- Service/query object methods
- Any method where the return type or parameter types are non-obvious

Skip annotations on Rails DSL (`validates`, `scope`, `belongs_to`, etc.) and trivial private helpers.

### Types & Annotations

No YARD annotations. No `@param`/`@return` tags. Use `# @rbs` instead (see above). For complex return values, a plain English comment is fine.

```ruby
# Returns the presigned download URL, or nil if no file is attached.
def download_url
  return unless file.attached?

  file.blob.service_url(expires_in: 1.hour)
end
```

---

## Rust



---

## Typescript

