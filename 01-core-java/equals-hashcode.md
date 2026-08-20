# equals() and hashCode() in Java

`equals()` and `hashCode()` are fundamental Java concepts and very common in backend interviews.

They are especially important when working with:

- HashMap
- HashSet
- HashTable
- Caches
- JPA entities
- DTOs
- Domain objects
- Collections

The key idea is:

```text
equals()
    ↓
Logical equality

hashCode()
    ↓
Hash-based lookup/bucketing
```

---

# 1. Object Class

Every Java class ultimately inherits from:

```java
java.lang.Object
```

`Object` provides methods including:

```java
equals()
hashCode()
toString()
getClass()
```

The two most important methods for hash-based collections are:

```java
equals()
hashCode()
```

---

# 2. What is equals()?

`equals()` determines whether two objects should be considered logically equal.

The default implementation in `Object` behaves like reference identity.

```java
public boolean equals(Object obj) {
    return this == obj;
}
```

Many classes override it to provide value-based equality.

For example, `String` compares its contents.

```java
String a = new String("Java");
String b = new String("Java");

System.out.println(a.equals(b));
```

Output:

```text
true
```

---

# 3. What is hashCode()?

`hashCode()` returns an integer hash value associated with an object.

```java
int hashCode()
```

Hash-based collections use hash codes to efficiently locate possible matching entries.

Examples:

```java
HashMap
HashSet
Hashtable
```

A hash code is **not a unique identifier**.

Different objects can have the same hash code.

This is called a collision.

---

# 4. The equals() / hashCode() Contract

This is the most important rule:

> If two objects are equal according to `equals()`, they must have the same hash code.

Formally:

```text
a.equals(b) == true
        ↓
a.hashCode() == b.hashCode()
```

The reverse is not guaranteed.

```text
a.hashCode() == b.hashCode()
        ↓
does NOT mean
        ↓
a.equals(b) == true
```

---

# 5. Why Can Different Objects Have the Same Hash Code?

Hash codes are integers.

There can be far more possible objects than possible `int` values.

Therefore collisions are unavoidable in general.

Example:

```text
Object A → hash 100
Object B → hash 100
```

The objects may still be different.

A HashMap handles this using equality checks after locating the relevant hash bucket.

---

# 6. Simple Example

```java
class User {

    private final int id;

    User(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object obj) {

        if (this == obj) {
            return true;
        }

        if (!(obj instanceof User other)) {
            return false;
        }

        return id == other.id;
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

Now:

```java
User user1 = new User(101);
User user2 = new User(101);
```

Then:

```java
user1.equals(user2)
```

is:

```text
true
```

and:

```java
user1.hashCode() == user2.hashCode()
```

is also:

```text
true
```

---

# 7. Why HashMap Needs Both

Consider:

```java
Map<User, String> users =
        new HashMap<>();

users.put(
    new User(101),
    "Sudhir"
);
```

Later:

```java
String name =
        users.get(new User(101));
```

For this to work correctly, `User` needs consistent `equals()` and `hashCode()` implementations.

The lookup roughly works like:

```text
key
 ↓
hashCode()
 ↓
find candidate bucket
 ↓
compare candidate keys using equals()
 ↓
return value
```

This is why both methods matter.

---

# 8. What Happens If hashCode() Is Not Overridden?

Suppose:

```java
class User {

    private final int id;

    User(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object obj) {

        if (this == obj) {
            return true;
        }

        if (!(obj instanceof User other)) {
            return false;
        }

        return id == other.id;
    }
}
```

Here `equals()` is overridden but `hashCode()` is not.

Two logically equal Users may have different identity-based hash codes.

This violates the contract.

Then:

```java
Map<User, String> map =
        new HashMap<>();

map.put(new User(101), "Sudhir");

System.out.println(
    map.get(new User(101))
);
```

may return:

```text
null
```

because the lookup can go to a different hash bucket.

---

# 9. What Happens If hashCode() Is Overridden but equals() Is Not?

This is also problematic.

Two objects can produce the same hash code but still not be logically equal.

The collection will use `equals()` to distinguish them.

So both methods need to agree on the same equality definition.

---

# 10. Rules of equals()

A correct `equals()` implementation should satisfy these general properties.

### Reflexive

```text
x.equals(x) == true
```

### Symmetric

```text
x.equals(y) == y.equals(x)
```

### Transitive

If:

```text
x.equals(y)
y.equals(z)
```

then:

```text
x.equals(z)
```

### Consistent

Repeated calls should return the same result as long as relevant state has not changed.

### Null

For a non-null reference:

```java
x.equals(null)
```

should return:

```text
false
```

---

# 11. Reflexive Example

Correct:

```java
user.equals(user)
```

must be:

```text
true
```

An implementation that returns false here violates the equals contract.

---

# 12. Symmetry Example

If:

```java
a.equals(b)
```

is true, then:

```java
b.equals(a)
```

must also be true.

Violating symmetry can cause very confusing collection behavior.

---

# 13. Transitivity Example

If:

```text
A equals B
B equals C
```

then:

```text
A equals C
```

must also be true.

This is especially important when designing equality across inheritance hierarchies.

---

# 14. Common equals() Implementation

A common pattern:

```java
@Override
public boolean equals(Object obj) {

    if (this == obj) {
        return true;
    }

    if (obj == null ||
        getClass() != obj.getClass()) {
        return false;
    }

    User other = (User) obj;

    return id == other.id;
}
```

Then:

```java
@Override
public int hashCode() {
    return Objects.hash(id);
}
```

This is a standard value-based implementation.

---

# 15. `instanceof` vs `getClass()` in equals()

Two common approaches are:

```java
obj instanceof User
```

and:

```java
getClass() == obj.getClass()
```

They are not always interchangeable.

### getClass()

Requires the exact same runtime class.

```java
if (obj == null ||
    getClass() != obj.getClass()) {
    return false;
}
```

### instanceof

Allows compatible subclass instances to participate.

```java
if (!(obj instanceof User other)) {
    return false;
}
```

Which approach is correct depends on the class hierarchy and equality semantics.

For non-final classes, equality across inheritance can become tricky and can violate symmetry/transitivity if designed carelessly.

---

# 16. Records and Equality

Java records automatically provide value-based implementations of:

```text
equals()
hashCode()
toString()
```

Example:

```java
public record User(
        Long id,
        String name
) {
}
```

Two records with the same component values compare equal.

```java
User a =
        new User(1L, "Sudhir");

User b =
        new User(1L, "Sudhir");

System.out.println(a.equals(b));
```

Output:

```text
true
```

---

# 17. Objects.equals()

Java provides:

```java
Objects.equals(a, b)
```

This is useful for null-safe equality comparison.

Conceptually:

```java
Objects.equals(a, b)
```

handles:

```text
both null → true
one null → false
both non-null → a.equals(b)
```

Example:

```java
String first = null;
String second = "Java";

boolean result =
        Objects.equals(first, second);
```

Result:

```text
false
```

---

# 18. Objects.hash()

Java provides:

```java
Objects.hash(...)
```

Example:

```java
@Override
public int hashCode() {

    return Objects.hash(
        id,
        name
    );
}
```

This is convenient when equality depends on multiple fields.

---

# 19. Multiple Fields in equals()

Suppose:

```java
class Employee {

    private final int id;
    private final String department;
}
```

If both fields define identity:

```java
@Override
public boolean equals(Object obj) {

    if (this == obj) {
        return true;
    }

    if (!(obj instanceof Employee other)) {
        return false;
    }

    return id == other.id &&
           Objects.equals(
               department,
               other.department
           );
}
```

Then hashCode must use the same equality-significant fields:

```java
@Override
public int hashCode() {

    return Objects.hash(
        id,
        department
    );
}
```

---

# 20. Important Rule: Same Fields

If `equals()` uses:

```text
id + department
```

then `hashCode()` should also use:

```text
id + department
```

Do not do:

```java
equals() → id + department
hashCode() → id
```

unless you intentionally design a hash function that still satisfies the contract. The safest normal approach is to use the same equality-significant fields.

---

# 21. HashSet Example

A `HashSet` uses equality and hashing to determine whether an element is already present.

```java
Set<User> users =
        new HashSet<>();

users.add(new User(101));
users.add(new User(101));
```

If equality is based on `id`, the Set should contain one logical User.

```text
size = 1
```

Without proper `equals()` and `hashCode()`, both objects may be treated as distinct.

---

# 22. HashMap Example

```java
Map<User, String> users =
        new HashMap<>();

User user1 = new User(101);

users.put(user1, "Sudhir");

User user2 = new User(101);

System.out.println(
    users.get(user2)
);
```

This works when:

```text
user1.equals(user2) == true
```

and:

```text
user1.hashCode() == user2.hashCode()
```

---

# 23. Mutable HashMap Keys

This is a major interview trap.

Avoid using mutable fields as equality/hash-code fields for keys in a HashMap.

Example:

```java
class User {

    int id;

    @Override
    public boolean equals(Object obj) {
        // based on id
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

Then:

```java
User user = new User(101);

map.put(user, "Sudhir");
```

If later:

```java
user.id = 999;
```

the key's hash code changes.

The HashMap still has the entry in the bucket corresponding to the old hash.

A lookup using the mutated key can fail.

---

# 24. Why Immutable Keys Are Safer

Consider:

```java
public final class UserId {

    private final long value;

    public UserId(long value) {
        this.value = value;
    }

    public long value() {
        return value;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof UserId other)) {
            return false;
        }

        return value == other.value;
    }

    @Override
    public int hashCode() {
        return Long.hashCode(value);
    }
}
```

Because the equality-significant state cannot change, it is safer as a map key.

---

# 25. HashMap Collision

A collision happens when different keys produce the same hash code.

Conceptually:

```text
Key A → hash 100
Key B → hash 100
```

Both may land in the same bucket.

HashMap then uses equality checks to determine whether the key is actually the same.

Modern Java HashMap can use tree structures for sufficiently collision-heavy buckets under appropriate conditions, improving worst-case lookup behavior compared with a long linked list.

Do not memorize implementation thresholds unless specifically asked; they are implementation details.

---

# 26. HashCode Should Be Efficient

A good hash function should distribute values reasonably across buckets.

Poor hashing can cause:

```text
many collisions
      ↓
more equality checks
      ↓
worse performance
```

Example of a poor implementation:

```java
@Override
public int hashCode() {
    return 1;
}
```

This technically can satisfy the equality contract, but causes excessive collisions.

---

# 27. Is Same Hash Code Enough for Equality?

No.

Example:

```text
Object A → hash 100
Object B → hash 100
```

They can still be different.

Therefore:

```text
same hash
    ≠
same object
```

The correct relationship is:

```text
equal objects
    ↓
same hash

same hash
    ↓
may or may not be equal
```

---

# 28. Is equals() Enough Without hashCode()?

No.

If objects are used in hash-based collections, overriding only `equals()` breaks the required contract.

Always override:

```java
equals()
hashCode()
```

together when implementing value-based equality.

---

# 29. `==` vs `equals()` vs `hashCode()`

| Operation | Purpose |
|---|---|
| `==` | Reference identity for objects |
| `equals()` | Logical equality |
| `hashCode()` | Hash value used by hash-based structures |

Example:

```java
User a = new User(101);
User b = new User(101);
```

Potentially:

```text
a == b
false

a.equals(b)
true

a.hashCode() == b.hashCode()
true
```

assuming equality/hashCode are implemented based on `id`.

---

# 30. JPA Entity Considerations

`equals()` and `hashCode()` become more subtle for JPA/Hibernate entities.

Example:

```java
@Entity
public class User {

    @Id
    @GeneratedValue
    private Long id;
}
```

A generated database ID may be `null` before persistence.

Therefore, blindly implementing equality based only on a generated ID can cause surprising behavior before the entity is persisted.

For JPA entities, equality should be designed according to the entity's identity model and Hibernate proxy considerations.

Do not automatically copy a generic `equals()` template into every entity.

---

# 31. DTO Equality

For DTOs and value objects, value-based equality is often straightforward.

Example:

```java
public record UserResponse(
        Long id,
        String name
) {
}
```

Records automatically provide appropriate component-based equality.

For a traditional DTO:

```java
class UserResponse {

    private Long id;
    private String name;

    // equals + hashCode
}
```

the fields included in equality should match the intended value semantics.

---

# 32. Entity vs Value Object

A useful design distinction:

### Entity

Identity is often more important than all attributes.

Example:

```text
User ID = 101
```

### Value Object

The value itself defines equality.

Example:

```text
Money(100, "INR")
```

Two equal Money values may represent the same value even if they are different objects.

This distinction helps decide what should participate in `equals()` and `hashCode()`.

---

# 33. Common Interview Question

### "What happens if two equal objects have different hash codes?"

That violates the `equals()`/`hashCode()` contract.

Hash-based collections may fail to find logically equal keys correctly.

---

# 34. Common Interview Question

### "Can two unequal objects have the same hash code?"

Yes.

That is a hash collision.

The contract only requires:

```text
equal → same hash
```

It does not require:

```text
same hash → equal
```

---

# 35. Common Interview Question

### "Why should hashCode be overridden when equals is overridden?"

Because hash-based collections use the hash code to locate a candidate bucket before using equality to distinguish keys.

If equal objects have different hash codes, an equal key may be searched in a different bucket and not found.

---

# 36. Common Interview Question

### "Can hashCode return a unique value for every object?"

Not in general.

`hashCode()` returns an `int`, so there are only a finite number of possible values.

Many distinct objects can therefore share the same hash code.

---

# 37. Common Interview Question

### "What happens if a HashMap key is modified after insertion?"

If the fields used by `equals()`/`hashCode()` change, the key's hash code can change.

The entry remains stored according to the old hash/bucket location.

A subsequent lookup may fail.

Therefore, keys should generally be immutable with respect to their equality/hash-code state.

---

# 38. Common Interview Question

### "Should every field be included in equals()?"

No.

Only fields that define the object's logical equality should be included.

For example:

```text
User
 ├── id
 ├── name
 ├── lastLoginTime
 └── temporarySessionState
```

It may make sense for equality to depend on:

```text
id
```

rather than mutable operational fields such as:

```text
lastLoginTime
temporarySessionState
```

The correct choice depends on the domain.

---

# 39. Coding Exercise — Correct User Equality

```java
import java.util.Objects;

public final class User {

    private final Long id;
    private final String name;

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    @Override
    public boolean equals(Object obj) {

        if (this == obj) {
            return true;
        }

        if (!(obj instanceof User other)) {
            return false;
        }

        return Objects.equals(id, other.id)
                && Objects.equals(name, other.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

---

# 40. Coding Exercise — HashSet Deduplication

```java
Set<User> users = new HashSet<>();

users.add(new User(1L, "Sudhir"));
users.add(new User(1L, "Sudhir"));

System.out.println(users.size());
```

If equality is based on both `id` and `name`:

```text
1
```

If `equals()`/`hashCode()` are not correctly implemented:

```text
2
```

may occur.

---

# 41. Coding Exercise — HashMap Lookup

```java
Map<User, String> map =
        new HashMap<>();

User first =
        new User(1L, "Sudhir");

map.put(first, "Developer");

User second =
        new User(1L, "Sudhir");

System.out.println(
    map.get(second)
);
```

Expected:

```text
Developer
```

when equality and hashing are consistent.

---

# 42. Lombok Consideration

In projects using Lombok, you may see:

```java
@EqualsAndHashCode
```

or:

```java
@Data
```

These can generate `equals()` and `hashCode()`.

However, do not blindly use generated equality for JPA entities.

Understand which fields participate in equality and how proxy/generated-ID behavior affects the entity.

---

# 43. Best Practices

### Do

- Override `equals()` and `hashCode()` together.
- Use the same equality-significant fields.
- Keep hash-code-relevant key state stable.
- Prefer immutable keys.
- Understand entity-specific equality rules.
- Use `Objects.equals()` for null-safe comparisons where appropriate.
- Use `Objects.hash()` for convenient multi-field hashing.

### Avoid

- Overriding only `equals()`.
- Using mutable equality fields as HashMap keys.
- Assuming same hash means objects are equal.
- Comparing String contents with `==`.
- Blindly generating equality for every JPA entity without considering identity semantics.

---

# 44. Quick Revision

```text
equals()
    ↓
Logical equality

hashCode()
    ↓
Hash value

equals true
    ↓
hashCode must be same

same hash
    ↓
Does NOT guarantee equality

HashMap
    ↓
hashCode → bucket
          ↓
       equals()

HashSet
    ↓
Uses hashing + equality

Mutable key
    ↓
Dangerous if equality state changes

Best practice
    ↓
Override equals() + hashCode() together
```

---

# 45. Strong Interview Answer

### Question: "Explain the equals() and hashCode() contract."

A strong concise answer:

> If two objects are equal according to `equals()`, they must return the same hash code. The reverse is not required because different objects can have the same hash code due to collisions. HashMap and HashSet use the hash code to locate a candidate bucket and then use equality to determine the actual match. That's why whenever I override `equals()`, I also override `hashCode()` using the same equality-significant fields.

---

# 46. Backend Interview Connection

For a Java backend developer, this topic is not just theoretical.

It appears in:

```text
HashMap
HashSet
Caching
DTO comparisons
Value objects
JPA/Hibernate entities
Deduplication
Collections
Database/domain modeling
```

A common real-world scenario is:

```text
Request
   ↓
DTO
   ↓
Domain object
   ↓
HashMap / Set / Cache
   ↓
equals + hashCode
```

Understanding these contracts helps prevent subtle production bugs involving missing map lookups, duplicate objects and unstable cache keys.
