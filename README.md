## UML Class Diagram

```mermaid
classDiagram
    class BPlusTree {
        -_filename: str
        -_tree_conf: TreeConf
        -_mem: FileMemory
        -_root_node_page: int
        -_is_open: bool
        +__init__(filename, page_size, order, key_size, value_size, cache_size, serializer)
        +insert(key, value, replace)
        +get(key, default)
        +delete(key)
        +close()
        +checkpoint()
        +__getitem__(item)
        +__setitem__(key, value)
        +__contains__(item)
        +__iter__()
        +items()
        +keys()
        +values()
        -_search_in_tree(key, node)
        -_split_leaf(old_node)
        -_split_parent(old_node)
        -_create_overflow(value)
        -_read_from_overflow(overflow_page)
    }

    class FileMemory {
        -_filename: str
        -_tree_conf: TreeConf
        -_lock: RWLock
        -_cache: LRUCache
        -_fd: FileIO
        -_wal: WAL
        +get_node(page)
        +set_node(node)
        +get_page(page)
        +set_page(page, data)
        +get_metadata()
        +set_metadata(root_node_page, tree_conf)
        +perform_checkpoint(reopen_wal)
        +close()
        +read_transaction
        +write_transaction
    }

    class WAL {
        -_filename: str
        -_page_size: int
        -_fd: FileIO
        -_not_committed_pages: dict
        -_committed_pages: dict
        +set_page(page, data)
        +get_page(page)
        +commit()
        +rollback()
        +checkpoint(data_file_fd, dir_fd)
        +close()
    }

    class Node {
        <<abstract>>
        #_tree_conf: TreeConf
        #entries: list
        #page: int
        #parent: Node
        #next_page: int
        +load(data)*
        +dump()*
        +insert_entry(entry)
        +remove_entry(key)
        +get_entry(key)
        +split_entries()
        +can_add_entry
        +can_delete_entry
        +num_children*
    }

    class LonelyRootNode {
        +convert_to_leaf()
    }

    class RootNode {
        +convert_to_internal()
    }

    class InternalNode {
    }

    class LeafNode {
    }

    class Entry {
        <<abstract>>
        +load(data)*
        +dump()*
        +__eq__(other)
        +__lt__(other)
    }

    class Record {
        +key
        +value: bytes
        +overflow_page: int
        +load(data)
        +dump()
    }

    class Reference {
        +key
        +before: int
        +after: int
        +load(data)
        +dump()
    }

    class Serializer {
        <<abstract>>
        +serialize(obj, key_size)*
        +deserialize(data)*
    }

    class IntSerializer {
        +serialize(obj, key_size)
        +deserialize(data)
    }

    class StrSerializer {
        +serialize(obj, key_size)
        +deserialize(data)
    }

    class UUIDSerializer {
        +serialize(obj, key_size)
        +deserialize(data)
    }

    class DatetimeUTCSerializer {
        +serialize(obj, key_size)
        +deserialize(data)
    }

    class TreeConf {
        <<namedtuple>>
        +page_size: int
        +order: int
        +key_size: int
        +value_size: int
        +serializer: Serializer
    }

    BPlusTree --> FileMemory : uses
    BPlusTree --> TreeConf : contains
    BPlusTree --> Node : manages
    FileMemory --> WAL : uses
    FileMemory --> TreeConf : uses
    FileMemory --> Node : loads/stores
    Node <|-- LonelyRootNode : inherits
    Node <|-- RootNode : inherits
    Node <|-- InternalNode : inherits
    Node <|-- LeafNode : inherits
    Node --> Entry : contains
    Entry <|-- Record : inherits
    Entry <|-- Reference : inherits
    LonelyRootNode --> Record : stores
    LeafNode --> Record : stores
    RootNode --> Reference : stores
    InternalNode --> Reference : stores
    Record --> Serializer : uses
    Reference --> Serializer : uses
    TreeConf --> Serializer : contains
    Serializer <|-- IntSerializer : inherits
    Serializer <|-- StrSerializer : inherits
    Serializer <|-- UUIDSerializer : inherits
    Serializer <|-- DatetimeUTCSerializer : inherits
```

## Package Relationship Diagram

This diagram shows the module dependencies and layered architecture:

```mermaid
graph TB
    subgraph "Public API Layer"
        init["__init__.py<br/>Exports: BPlusTree, Serializers"]
    end

    subgraph "Core Logic Layer"
        tree["tree.py<br/>BPlusTree class"]
    end

    subgraph "Data Structure Layer"
        node["node.py<br/>Node hierarchy"]
        entry["entry.py<br/>Entry types"]
    end

    subgraph "Type System Layer"
        serializer["serializer.py<br/>Serializer implementations"]
        const["const.py<br/>TreeConf, constants"]
    end

    subgraph "Persistence Layer"
        memory["memory.py<br/>FileMemory, WAL"]
    end

    subgraph "Utilities Layer"
        utils["utils.py<br/>Helper functions"]
    end

    subgraph "External Dependencies"
        rwlock["rwlock<br/>Reader-writer locks"]
        cachetools["cachetools<br/>LRU cache"]
        temporenc["temporenc<br/>Datetime encoding"]
    end

    init --> tree
    init --> serializer
    tree --> node
    tree --> entry
    tree --> memory
    tree --> const
    tree --> utils
    node --> entry
    node --> const
    entry --> const
    entry --> serializer
    memory --> const
    memory --> node
    serializer --> const
    memory --> rwlock
    memory --> cachetools
    serializer --> temporenc
```
