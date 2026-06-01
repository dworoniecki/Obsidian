---
tags: [cheat-sheet, python, standard-library]
---

# Python Standard Library Cheatsheet

Quick reference for commonly used modules that ship with Python — no install required.

## Contents

- [[#Common Workflows]]
- [[#collections]]
- [[#csv]]
- [[#datetime]]
- [[#pathlib]]
- [[#dataclasses]]
- [[#itertools]]
- [[#json]]
- [[#logging]]
- [[#os]]
- [[#re]]
- [[#random]]
- [[#shutil]]
- [[#sys]]
- [[#types]]
- [[#decimal]]
- [[#io]]
- [[#subprocess]]
- [[#functools]]
- [[#enum]]
- [[#statistics]]
- [[#time]]
- [[#math]]

---

## Common Workflows

### Load Config from Environment + JSON
```python
import json, os
from pathlib import Path

env = os.environ.get('APP_ENV', 'dev')
config_path = Path(f'config/{env}.json')

if config_path.exists():
    config = json.loads(config_path.read_text())
else:
    config = {}
```

### Process CSV → JSON with Logging
```python
import csv, json, logging
from pathlib import Path

logging.basicConfig(level=logging.INFO, format='%(levelname)s %(message)s')
logger = logging.getLogger(__name__)

input_path = Path('data/loans.csv')
output_path = Path('data/loans.json')

rows = []
with input_path.open(newline='') as f:
    reader = csv.DictReader(f)
    for i, row in enumerate(reader):
        rows.append(row)

logger.info("Read %d rows from %s", len(rows), input_path)
output_path.write_text(json.dumps(rows, indent=2))
logger.info("Wrote %s", output_path)
```

### CLI Script with sys.argv
```python
import sys
from pathlib import Path

def main():
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <input> <output>", file=sys.stderr)
        sys.exit(1)

    src, dst = Path(sys.argv[1]), Path(sys.argv[2])
    if not src.exists():
        print(f"Error: {src} not found", file=sys.stderr)
        sys.exit(1)

    # ... do work ...

if __name__ == '__main__':
    main()
```

### Archive and Rotate Output Files
```python
import shutil
from datetime import date
from pathlib import Path

output_dir = Path('reports')
archive_name = f"reports_{date.today().strftime('%Y%m%d')}"

shutil.make_archive(archive_name, 'zip', output_dir)
shutil.rmtree(output_dir)
output_dir.mkdir()
print(f"Archived to {archive_name}.zip")
```

---

## collections

Specialized container types beyond the built-in `list`, `dict`, `set`.

```python
from collections import Counter, defaultdict, OrderedDict, namedtuple, deque
```

| Class / Function | Description | Example | Use Case |
|------------------|-------------|---------|----------|
| `Counter(iterable)` | Count occurrences of elements | `Counter(['a','b','a'])` → `{'a':2,'b':1}` | Frequency counts |
| `.most_common(n)` | Top n most frequent items | `Counter(words).most_common(5)` | Top values in a column |
| `defaultdict(type)` | Dict that auto-creates missing keys | `defaultdict(list)` | Group rows by key |
| `OrderedDict()` | Dict that remembers insertion order | `OrderedDict([('a',1),('b',2)])` | Pre-3.7 ordered dict |
| `namedtuple(name, fields)` | Tuple subclass with named fields | `Point = namedtuple('Point', ['x','y'])` | Lightweight data record |
| `deque(iterable)` | Double-ended queue | `deque([1,2,3], maxlen=100)` | Efficient append/pop from both ends |

```python
# Counter — frequency table
from collections import Counter
statuses = ['open', 'closed', 'open', 'pending', 'closed', 'closed']
counts = Counter(statuses)
# Counter({'closed': 3, 'open': 2, 'pending': 1})
counts.most_common(2)  # [('closed', 3), ('open', 2)]

# defaultdict — group records by key
from collections import defaultdict
grouped = defaultdict(list)
for row in rows:
    grouped[row['category']].append(row)

# namedtuple — structured record
from collections import namedtuple
Loan = namedtuple('Loan', ['id', 'balance', 'status'])
loan = Loan(id='L001', balance=15000, status='open')
loan.balance  # 15000
```

---

## csv

Read and write CSV files.

```python
import csv
```

| Function / Class | Description | Key Args | Example | Use Case |
|------------------|-------------|----------|---------|----------|
| `csv.reader(f)` | Read CSV row by row as lists | `delimiter`, `quotechar` | `csv.reader(open('data.csv'))` | Iterate rows |
| `csv.writer(f)` | Write rows as CSV | `delimiter`, `quotechar` | `csv.writer(open('out.csv','w'))` | Write to CSV |
| `csv.DictReader(f)` | Read rows as dicts (header as keys) | `fieldnames` | `csv.DictReader(open('data.csv'))` | Named column access |
| `csv.DictWriter(f, fieldnames)` | Write dicts as CSV rows | `fieldnames`, `extrasaction` | `csv.DictWriter(f, fields)` | Write dicts to CSV |
| `.writerow(row)` | Write a single row | N/A | `writer.writerow(['a','b'])` | Write one row |
| `.writerows(rows)` | Write multiple rows at once | N/A | `writer.writerows(data)` | Bulk write |
| `open(f, encoding='utf-8-sig')` | Strip BOM from Excel-exported CSVs | N/A | `open('data.csv', encoding='utf-8-sig')` | Fix garbled first column header from Excel |
| `csv.Sniffer().sniff(sample)` | Auto-detect delimiter and quoting style | N/A | `csv.Sniffer().sniff(f.read(1024))` | Unknown or inconsistent delimiter |

```python
import csv

# Read with DictReader
with open('loans.csv', newline='') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['loan_id'], row['balance'])  # Access by column name

# Write with DictWriter
fields = ['loan_id', 'balance', 'status']
with open('output.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=fields)
    writer.writeheader()
    writer.writerows([
        {'loan_id': 'L001', 'balance': 15000, 'status': 'open'},
        {'loan_id': 'L002', 'balance': 9500,  'status': 'closed'},
    ])

# Handle non-standard delimiters
with open('data.tsv', newline='') as f:
    reader = csv.reader(f, delimiter='\t')
    rows = list(reader)

# Handle encoding — 'utf-8-sig' strips the BOM that Excel adds to CSVs
# Without it, the first column header appears as '\ufeffcolumn_name'
with open('export.csv', newline='', encoding='utf-8-sig') as f:
    reader = csv.DictReader(f)
    rows = list(reader)

# Auto-detect delimiter with Sniffer (useful for vendor files)
with open('mystery.csv', newline='') as f:
    sample = f.read(1024)
    dialect = csv.Sniffer().sniff(sample)
    f.seek(0)
    reader = csv.reader(f, dialect)
    rows = list(reader)
```

---

## datetime

Work with dates, times, and durations.

```python
from datetime import date, time, datetime, timedelta, timezone
```

| Class / Method | Description | Example | Use Case |
|----------------|-------------|---------|----------|
| `date.today()` | Today's date (no time) | `date.today()` → `2024-11-15` | Current date |
| `datetime.now()` | Current local date and time | `datetime.now()` | Timestamp now |
| `datetime.utcnow()` | Current UTC date and time | `datetime.utcnow()` | UTC timestamp |
| `datetime(y, m, d, h, min, s)` | Create a specific datetime | `datetime(2024, 1, 15, 9, 30)` | Fixed timestamp |
| `timedelta(days=n)` | Duration / offset | `timedelta(days=30)` | Date arithmetic |
| `.strftime(fmt)` | Format datetime to string | `dt.strftime('%Y-%m-%d')` | Format for output |
| `datetime.strptime(s, fmt)` | Parse string to datetime | `datetime.strptime('2024-01-15', '%Y-%m-%d')` | Parse date strings |
| `.isoformat()` | ISO 8601 string | `datetime.now().isoformat()` | Standard string format |
| `datetime.fromisoformat(s)` | Parse ISO 8601 string | `datetime.fromisoformat('2024-01-15')` | Parse ISO strings |
| `.replace(...)` | Return copy with fields changed | `dt.replace(hour=0, minute=0)` | Truncate to day |
| `.timestamp()` | Unix epoch float | `datetime.now().timestamp()` | Epoch seconds |
| `datetime.fromtimestamp(ts)` | Datetime from epoch | `datetime.fromtimestamp(1700000000)` | Convert epoch |
| `datetime.now(timezone.utc)` | Timezone-aware UTC datetime (preferred) | `datetime.now(timezone.utc)` | Correct UTC — avoids naive datetime bug |
| `ZoneInfo('Region/City')` | IANA timezone object (3.9+) | `ZoneInfo('America/New_York')` | Localize to a named timezone |
| `.astimezone(tz)` | Convert aware datetime to another timezone | `dt.astimezone(timezone.utc)` | Normalize to UTC for storage |
| `.date()` | Extract date part from a datetime | `datetime.now().date()` | Strip the time portion |

```python
from datetime import date, datetime, timedelta

# Date arithmetic
today = date.today()
thirty_days_ago = today - timedelta(days=30)
due_date = today + timedelta(weeks=2)

# Parse and format
raw = '2024-01-15 09:30:00'
dt = datetime.strptime(raw, '%Y-%m-%d %H:%M:%S')
formatted = dt.strftime('%B %d, %Y')  # 'January 15, 2024'

# Compare dates
if dt.date() < date.today():
    print("Overdue")

# Days between two dates
delta = date(2024, 12, 31) - date.today()
print(f"{delta.days} days until year end")

# ⚠️  datetime.utcnow() returns a NAIVE datetime — no timezone info attached.
#    This looks like UTC but has no tzinfo, which causes bugs when comparing
#    with aware datetimes. Use datetime.now(timezone.utc) instead.
datetime.utcnow()                # naive — avoid (deprecated in 3.12)
datetime.now(timezone.utc)       # aware UTC — preferred

# Timezone-aware datetimes (Python 3.9+)
from zoneinfo import ZoneInfo
eastern = ZoneInfo('America/New_York')
now_eastern = datetime.now(eastern)
now_utc = now_eastern.astimezone(timezone.utc)
print(now_utc.isoformat())       # '2024-01-15T14:30:00+00:00'

# First and last day of the current month
today = date.today()
first_of_month = today.replace(day=1)
next_month = (first_of_month + timedelta(days=32)).replace(day=1)
last_of_month = next_month - timedelta(days=1)
```

**Common format codes:** `%Y` year, `%m` month, `%d` day, `%H` hour (24h), `%M` minute, `%S` second, `%A` weekday name, `%B` month name

---

## pathlib

Object-oriented filesystem paths. Prefer over `os.path` for new code.

```python
from pathlib import Path
```

| Method / Property | Description | Example | Use Case |
|-------------------|-------------|---------|----------|
| `Path('path/to/file')` | Create a Path object | `Path('data/loans.csv')` | Start here |
| `Path.cwd()` | Current working directory | `Path.cwd()` | Where am I? |
| `Path.home()` | User home directory | `Path.home() / 'data'` | Home-relative paths |
| `p / 'subdir'` | Join paths with `/` operator | `base / 'raw' / 'file.csv'` | Build paths cleanly |
| `p.exists()` | Check if path exists | `Path('data.csv').exists()` | Guard before reading |
| `p.is_file()` / `p.is_dir()` | Check type | `p.is_dir()` | File vs directory check |
| `p.mkdir(parents=True, exist_ok=True)` | Create directory | `Path('out/raw').mkdir(parents=True, exist_ok=True)` | Create nested dirs |
| `p.read_text()` / `p.write_text(s)` | Read/write file as string | `Path('config.json').read_text()` | Simple file I/O |
| `p.read_bytes()` / `p.write_bytes(b)` | Read/write file as bytes | `p.read_bytes()` | Binary file I/O |
| `p.glob('*.csv')` | Find matching files | `list(Path('data').glob('*.csv'))` | List files by pattern |
| `p.rglob('*.py')` | Recursive glob | `list(Path('.').rglob('*.py'))` | Find files in subdirs |
| `p.stem` | Filename without extension | `Path('data.csv').stem` → `'data'` | Strip extension |
| `p.suffix` | File extension | `Path('data.csv').suffix` → `'.csv'` | Get extension |
| `p.name` | Filename with extension | `Path('a/b/data.csv').name` → `'data.csv'` | Just the filename |
| `p.parent` | Parent directory | `Path('a/b/file.csv').parent` → `Path('a/b')` | Go up one level |
| `p.rename(new)` | Rename / move file | `p.rename(p.with_suffix('.parquet'))` | Rename file |
| `p.unlink()` | Delete a file | `Path('temp.csv').unlink()` | Delete file |
| `p.with_suffix(ext)` | Copy path with new extension | `p.with_suffix('.json')` | Change extension |
| `p.stat().st_size` | File size in bytes | `p.stat().st_size` | Check file size |

```python
from pathlib import Path

# Build and create directory structure
base = Path('data')
raw_dir = base / 'raw'
processed_dir = base / 'processed'
raw_dir.mkdir(parents=True, exist_ok=True)
processed_dir.mkdir(parents=True, exist_ok=True)

# Find all CSVs and process them
for csv_file in Path('data/raw').glob('*.csv'):
    print(csv_file.stem)                  # filename without extension
    out = Path('data/processed') / csv_file.with_suffix('.parquet').name

# Read / write text
config = Path('config.json').read_text()
Path('output.txt').write_text('done\n')
```

---

## dataclasses

Reduce boilerplate for data-holding classes.

```python
from dataclasses import dataclass, field, asdict, astuple, fields
```

| Decorator / Function | Description | Example | Use Case |
|----------------------|-------------|---------|----------|
| `@dataclass` | Auto-generate `__init__`, `__repr__`, `__eq__` | `@dataclass` | Simple data containers |
| `@dataclass(frozen=True)` | Immutable dataclass (hashable) | `@dataclass(frozen=True)` | Use as dict key or in sets |
| `@dataclass(order=True)` | Generate comparison methods (`<`, `>`) | `@dataclass(order=True)` | Sortable records |
| `field(default_factory=list)` | Mutable default values | `items: list = field(default_factory=list)` | Default list/dict |
| `field(repr=False)` | Exclude field from `__repr__` | `password: str = field(repr=False)` | Hide sensitive fields |
| `asdict(obj)` | Convert dataclass to dict | `asdict(loan)` | Serialize to dict / JSON |
| `astuple(obj)` | Convert dataclass to tuple | `astuple(loan)` | Unpack into tuple |
| `fields(obj)` | Get field metadata | `fields(Loan)` | Introspect field names/types |
| `__post_init__(self)` | Called after `__init__` — validate or compute fields | see example | Validate inputs, set derived fields |
| `field(init=False)` | Exclude from constructor — set in `__post_init__` | `fee: float = field(init=False)` | Computed/derived fields |

```python
from dataclasses import dataclass, field, asdict

@dataclass
class Loan:
    id: str
    balance: float
    status: str = 'open'
    tags: list = field(default_factory=list)

loan = Loan(id='L001', balance=15000.0)
print(loan)              # Loan(id='L001', balance=15000.0, status='open', tags=[])
asdict(loan)             # {'id': 'L001', 'balance': 15000.0, 'status': 'open', 'tags': []}

# Frozen (immutable + hashable)
@dataclass(frozen=True)
class Point:
    x: float
    y: float

points = {Point(0, 0), Point(1, 2)}  # Can be used in a set

# __post_init__ — validate inputs and compute derived fields
@dataclass
class Transaction:
    amount: float
    currency: str
    fee_pct: float = 0.02
    fee: float = field(init=False)   # not passed by caller — computed below

    def __post_init__(self):
        if self.amount <= 0:
            raise ValueError(f"Amount must be positive, got {self.amount}")
        self.fee = round(self.amount * self.fee_pct, 2)

t = Transaction(amount=1000.0, currency='USD')
t.fee   # 20.0
```

---

## itertools

Memory-efficient iterators for looping patterns.

```python
import itertools
```

| Function | Description | Example | Use Case |
|----------|-------------|---------|----------|
| `chain(*iterables)` | Flatten multiple iterables into one | `chain([1,2], [3,4])` → `1,2,3,4` | Combine lists without copying |
| `chain.from_iterable(it)` | Flatten one iterable of iterables | `chain.from_iterable([[1,2],[3,4]])` | Flatten list of lists |
| `islice(it, n)` | Take first n items from iterator | `islice(reader, 100)` | Preview large iterators |
| `islice(it, start, stop)` | Slice an iterator | `islice(it, 10, 20)` | Skip and take |
| `groupby(it, key)` | Group consecutive equal-key items | `groupby(sorted_rows, key=lambda r: r['date'])` | Group sorted data |
| `product(*iterables)` | Cartesian product | `product([1,2], ['a','b'])` | All combinations |
| `combinations(it, r)` | r-length combos (no repeats) | `combinations([1,2,3], 2)` | Pairs without order |
| `permutations(it, r)` | r-length ordered arrangements | `permutations([1,2,3], 2)` | Ordered arrangements |
| `batched(it, n)` | Split into chunks of size n (3.12+) | `batched(rows, 1000)` | Batch processing |
| `accumulate(it)` | Running total | `accumulate([1,2,3])` → `1,3,6` | Cumulative sum |
| `takewhile(pred, it)` | Take while condition is true | `takewhile(lambda x: x < 5, it)` | Stop on condition |
| `dropwhile(pred, it)` | Skip while condition is true | `dropwhile(lambda x: x < 5, it)` | Skip leading values |
| `zip_longest(*its, fillvalue)` | Zip with fill for unequal lengths | `zip_longest(a, b, fillvalue=None)` | Zip unequal lists |
| `repeat(val, n)` | Repeat a value n times | `repeat(0, 5)` → `0,0,0,0,0` | Default fill values |
| `count(start, step)` | Infinite counter | `count(1, 2)` → `1,3,5,...` | Auto-incrementing index |

```python
import itertools

# Flatten a list of lists
nested = [[1, 2], [3, 4], [5]]
flat = list(itertools.chain.from_iterable(nested))  # [1, 2, 3, 4, 5]

# Group by a key (data MUST be sorted by key first)
from itertools import groupby
rows = sorted(data, key=lambda r: r['status'])
for status, group in groupby(rows, key=lambda r: r['status']):
    items = list(group)
    print(f"{status}: {len(items)}")

# Batch rows for bulk insert (Python 3.12+)
from itertools import batched
for batch in batched(all_rows, 500):
    db.insert_many(batch)

# Pre-3.12 batch workaround
def chunked(it, n):
    it = iter(it)
    while chunk := list(itertools.islice(it, n)):
        yield chunk
```

---

## json

Serialize and deserialize JSON data.

```python
import json
```

| Function | Description | Key Args | Example | Use Case |
|----------|-------------|----------|---------|----------|
| `json.dumps(obj)` | Object → JSON string | `indent`, `sort_keys`, `default` | `json.dumps({'a': 1})` | Serialize to string |
| `json.loads(s)` | JSON string → Python object | N/A | `json.loads('{"a": 1}')` | Deserialize string |
| `json.dump(obj, f)` | Object → JSON file | `indent`, `sort_keys`, `default` | `json.dump(data, open('out.json','w'))` | Write JSON file |
| `json.load(f)` | JSON file → Python object | N/A | `json.load(open('config.json'))` | Read JSON file |
| `indent=2` | Pretty-print with indentation | N/A | `json.dumps(data, indent=2)` | Human-readable output |
| `default=str` | Fallback serializer for unknown types | N/A | `json.dumps(data, default=str)` | Handle dates, decimals |
| `sort_keys=True` | Sort dict keys alphabetically | N/A | `json.dumps(data, sort_keys=True)` | Deterministic output |
| `json.JSONDecodeError` | Exception raised for invalid/malformed JSON | N/A | `except json.JSONDecodeError as e:` | Handle bad JSON from external sources |
| `object_hook=func` | Call func on every decoded JSON object | N/A | `json.loads(s, object_hook=fn)` | Auto-convert types (dates, Decimals) on load |

```python
import json

# Read config file
with open('config.json') as f:
    config = json.load(f)

# Write pretty JSON
with open('results.json', 'w') as f:
    json.dump(results, f, indent=2)

# Handle non-serializable types (datetime, Decimal)
from datetime import datetime
from decimal import Decimal
data = {'date': datetime.now(), 'amount': Decimal('15000.00')}

json.dumps(data, default=str)   # Converts everything to string as fallback

# Custom serializer
def serialize(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Not serializable: {type(obj)}")

json.dumps(data, default=serialize)

# Round-trip a dict through JSON (deep copy + validation)
clean = json.loads(json.dumps(original))

# Always handle JSONDecodeError when parsing external/untrusted data
try:
    data = json.loads(raw_string)
except json.JSONDecodeError as e:
    print(f"Invalid JSON at line {e.lineno}, col {e.colno}: {e.msg}")
    raise

# JSON Lines (NDJSON) — one JSON object per line, common in event pipelines
# Read
with open('events.jsonl') as f:
    records = [json.loads(line) for line in f if line.strip()]

# Write
with open('output.jsonl', 'w') as f:
    for record in records:
        f.write(json.dumps(record) + '\n')
```

---

## logging

Production-quality logging — prefer over `print()`.

```python
import logging
```

| Function / Method | Description | Example | Use Case |
|-------------------|-------------|---------|----------|
| `logging.basicConfig(...)` | Configure root logger | `logging.basicConfig(level=logging.INFO)` | Quick setup |
| `logging.getLogger(name)` | Get/create a named logger | `logging.getLogger(__name__)` | Per-module logger |
| `logger.debug(msg)` | Debug message (level 10) | `logger.debug("Processing row %s", id)` | Verbose dev info |
| `logger.info(msg)` | Info message (level 20) | `logger.info("Loaded %d rows", n)` | General progress |
| `logger.warning(msg)` | Warning (level 30) | `logger.warning("Missing value in row %s", id)` | Non-fatal issues |
| `logger.error(msg)` | Error (level 40) | `logger.error("Failed to connect: %s", e)` | Recoverable errors |
| `logger.exception(msg)` | Error + full traceback | `logger.exception("Unexpected error")` | Inside except blocks |
| `logger.critical(msg)` | Critical (level 50) | `logger.critical("DB unreachable")` | Fatal issues |
| `logging.FileHandler(path)` | Write logs to a file | `FileHandler('app.log')` | Persistent log file |
| `logging.StreamHandler()` | Write logs to console | `StreamHandler()` | Console output |
| `logging.handlers.RotatingFileHandler(path, maxBytes, backupCount)` | Rotate log file when it reaches a size limit | `RotatingFileHandler('app.log', maxBytes=10**7, backupCount=5)` | Long-running pipelines |
| `logging.LoggerAdapter(logger, extra)` | Wrap logger to inject context into every message | `LoggerAdapter(logger, {'run_id': run_id})` | Tag all logs with pipeline/run context |

```python
import logging

# Standard module-level setup (recommended pattern)
logger = logging.getLogger(__name__)

# Configure at application entry point (once)
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s %(name)s %(levelname)s %(message)s',
    handlers=[
        logging.StreamHandler(),             # Console
        logging.FileHandler('pipeline.log'), # File
    ]
)

# Use % formatting (lazy — no string build if level filtered out)
logger.info("Processing file: %s", filepath)
logger.warning("Row %d skipped: missing balance", row_num)

# Log exceptions with traceback
try:
    result = risky_operation()
except Exception:
    logger.exception("Operation failed")   # Includes full traceback
    raise

# RotatingFileHandler — rotate at 10MB, keep 5 backups
from logging.handlers import RotatingFileHandler
handler = RotatingFileHandler('pipeline.log', maxBytes=10_000_000, backupCount=5)
handler.setFormatter(logging.Formatter('%(asctime)s %(levelname)s %(message)s'))
logging.getLogger().addHandler(handler)

# LoggerAdapter — inject run context into every log line automatically
# Avoids manually passing run_id to every logger.info() call
adapter = logging.LoggerAdapter(logger, {'run_id': 'run_20240115_001'})
logging.basicConfig(format='%(asctime)s [%(run_id)s] %(levelname)s %(message)s')
adapter.info("Starting extraction")   # → 2024-01-15 09:00:00 [run_20240115_001] INFO Starting extraction
adapter.error("Failed on row 42")     # → 2024-01-15 09:00:01 [run_20240115_001] ERROR Failed on row 42
```

---

## os

Operating system interface — environment variables, paths, process info.

```python
import os
```

| Function / Attribute | Description | Example | Use Case |
|----------------------|-------------|---------|----------|
| `os.getcwd()` | Current working directory | `os.getcwd()` | Where am I? |
| `os.chdir(path)` | Change working directory | `os.chdir('/data')` | Change directory |
| `os.listdir(path)` | List directory contents | `os.listdir('.')` | List files |
| `os.makedirs(path, exist_ok=True)` | Create directories recursively | `os.makedirs('a/b/c', exist_ok=True)` | Create nested dirs |
| `os.remove(path)` | Delete a file | `os.remove('temp.csv')` | Delete file |
| `os.rename(src, dst)` | Rename / move a file | `os.rename('old.csv', 'new.csv')` | Rename |
| `os.path.join(a, b)` | Join path components | `os.path.join('data', 'file.csv')` | Build paths (use `pathlib` instead) |
| `os.path.exists(path)` | Check if path exists | `os.path.exists('config.json')` | Guard before reading |
| `os.path.isfile(path)` / `os.path.isdir(path)` | Check file vs dir | `os.path.isdir('data/')` | Type check |
| `os.path.basename(path)` | Filename from path | `os.path.basename('/a/b/file.csv')` → `'file.csv'` | Extract filename |
| `os.path.dirname(path)` | Directory from path | `os.path.dirname('/a/b/file.csv')` → `'/a/b'` | Extract directory |
| `os.environ` | Dict of environment variables | `os.environ['HOME']` | Read env vars |
| `os.environ.get(key, default)` | Safe env var read | `os.environ.get('DB_URL', 'localhost')` | Env var with fallback |
| `os.getenv(key, default)` | Same as `environ.get` | `os.getenv('SECRET_KEY')` | Read env var |
| `os.cpu_count()` | Number of CPU cores | `os.cpu_count()` | Set parallelism |
| `os.walk(path)` | Walk directory tree | `for root, dirs, files in os.walk('.')` | Recurse directories |

```python
import os

# Environment variables (config, secrets)
db_url = os.environ.get('DATABASE_URL', 'sqlite:///local.db')
api_key = os.environ.get('API_KEY')
if not api_key:
    raise ValueError("API_KEY not set")

# Walk directory
for root, dirs, files in os.walk('data/'):
    for fname in files:
        full_path = os.path.join(root, fname)
        print(full_path)

# Prefer pathlib for new path work, but os.environ is still the standard way
# to read environment variables
```

> **Tip:** Prefer `pathlib.Path` over `os.path` for file/directory operations. Use `os` mainly for environment variables and process-level operations.

---

## re

Regular expressions for pattern matching and text manipulation.

```python
import re
```

| Function | Description | Key Args | Example | Use Case |
|----------|-------------|----------|---------|----------|
| `re.match(pattern, s)` | Match at start of string | `flags` | `re.match(r'\d+', '123abc')` | Check prefix |
| `re.search(pattern, s)` | Find first match anywhere | `flags` | `re.search(r'\d+', 'abc123')` | Find anywhere |
| `re.findall(pattern, s)` | Return all matches as list | `flags` | `re.findall(r'\d+', 'a1 b2 c3')` | Get all matches |
| `re.finditer(pattern, s)` | Return iterator of match objects | `flags` | `re.finditer(r'\w+', text)` | Iterate matches |
| `re.sub(pattern, repl, s)` | Replace matches | `count`, `flags` | `re.sub(r'\s+', '_', 'a b c')` | Find and replace |
| `re.split(pattern, s)` | Split string on pattern | `maxsplit` | `re.split(r'[,;]', 'a,b;c')` | Split on multiple delimiters |
| `re.compile(pattern)` | Pre-compile a pattern | `flags` | `pat = re.compile(r'\d+')` | Reuse pattern in loops |
| `match.group(0)` | Full match string | N/A | `m.group(0)` | Get matched text |
| `match.group(n)` | Nth capture group | N/A | `m.group(1)` | Get captured group |
| `match.groups()` | All capture groups as tuple | N/A | `m.groups()` | All groups at once |

**Common flags:** `re.IGNORECASE` / `re.I`, `re.MULTILINE` / `re.M`, `re.DOTALL` / `re.S`

```python
import re

# Extract numbers from a string
re.findall(r'\d+\.?\d*', 'balance: $15,000.50')  # ['15', '000.50']

# Validate email (basic)
pattern = re.compile(r'^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$')
pattern.match('user@example.com') is not None  # True

# Named capture groups
m = re.search(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})', '2024-01-15')
m.group('year')   # '2024'
m.group('month')  # '01'

# Replace with function
def normalize(m):
    return m.group(0).upper()
re.sub(r'\b[a-z]+\b', normalize, 'hello world')  # 'HELLO WORLD'

# Compile for reuse in loops (faster)
loan_id_re = re.compile(r'^L\d{3,}$')
valid = [id for id in ids if loan_id_re.match(id)]
```

---

## random

Generate random numbers and make random selections.

```python
import random
```

| Function | Description | Example | Use Case |
|----------|-------------|---------|----------|
| `random.random()` | Float in `[0.0, 1.0)` | `random.random()` | Probability check |
| `random.randint(a, b)` | Integer in `[a, b]` inclusive | `random.randint(1, 100)` | Random integer |
| `random.uniform(a, b)` | Float in `[a, b]` | `random.uniform(0.5, 1.5)` | Random float in range |
| `random.choice(seq)` | Random element from sequence | `random.choice(['a','b','c'])` | Pick one item |
| `random.choices(seq, k=n)` | n random elements (with replacement) | `random.choices(items, k=5)` | Sample with replacement |
| `random.sample(seq, k)` | k unique elements (no replacement) | `random.sample(users, 100)` | Random subset |
| `random.shuffle(lst)` | Shuffle list in place | `random.shuffle(rows)` | Randomize order |
| `random.seed(n)` | Set seed for reproducibility | `random.seed(42)` | Reproducible results |
| `random.gauss(mu, sigma)` | Gaussian (normal) distribution | `random.gauss(0, 1)` | Simulated data |

```python
import random

random.seed(42)  # Always set for reproducibility

# Sample rows for testing (no replacement)
sample = random.sample(all_rows, k=1000)

# Shuffle in place
random.shuffle(data)
train, test = data[:800], data[800:]

# Weighted random choice
options = ['A', 'B', 'C']
weights = [0.7, 0.2, 0.1]
random.choices(options, weights=weights, k=10)

# Simulate a value
simulated_balance = random.uniform(1000, 50000)
```

---

## shutil

High-level file and directory operations (copy, move, archive).

```python
import shutil
```

| Function | Description | Key Args | Example | Use Case |
|----------|-------------|----------|---------|----------|
| `shutil.copy(src, dst)` | Copy file (no metadata) | N/A | `shutil.copy('a.csv', 'backup/a.csv')` | Copy file |
| `shutil.copy2(src, dst)` | Copy file + preserve metadata | N/A | `shutil.copy2('a.csv', 'backup/')` | Copy with timestamps |
| `shutil.copytree(src, dst)` | Copy entire directory tree | `dirs_exist_ok` (3.8+) | `shutil.copytree('src/', 'dst/')` | Copy directory |
| `shutil.move(src, dst)` | Move file or directory | N/A | `shutil.move('raw/file.csv', 'processed/')` | Move to new location |
| `shutil.rmtree(path)` | Delete directory tree | `ignore_errors` | `shutil.rmtree('tmp/')` | Delete folder recursively |
| `shutil.make_archive(name, fmt, root)` | Create zip/tar archive | `format` (zip, tar, gztar) | `shutil.make_archive('backup', 'zip', 'data/')` | Archive a directory |
| `shutil.unpack_archive(file, dst)` | Extract archive | `format` | `shutil.unpack_archive('data.zip', 'out/')` | Extract archive |
| `shutil.disk_usage(path)` | Disk usage stats | N/A | `shutil.disk_usage('/')` | Check disk space |
| `shutil.which(cmd)` | Find path to an executable | N/A | `shutil.which('python')` | Locate command |

```python
import shutil
from pathlib import Path

# Copy a file to a backup directory
shutil.copy2('data/loans.csv', 'backup/')

# Move processed files
shutil.move('data/raw/file.csv', 'data/processed/file.csv')

# Create a zip archive of a directory
shutil.make_archive('data_backup_2024', 'zip', 'data/')
# → data_backup_2024.zip

# Extract an archive
shutil.unpack_archive('archive.tar.gz', 'extracted/')

# Delete a temp directory
shutil.rmtree('tmp/', ignore_errors=True)

# Check disk space
total, used, free = shutil.disk_usage('/')
print(f"Free: {free // (2**30)} GB")
```

---

## sys

System-specific parameters and functions.

```python
import sys
```

| Attribute / Function | Description | Example | Use Case |
|----------------------|-------------|---------|----------|
| `sys.argv` | Command-line arguments as list | `sys.argv[1]` | Read CLI arguments |
| `sys.exit(code)` | Exit with status code | `sys.exit(1)` | Exit on error |
| `sys.stdin` / `sys.stdout` / `sys.stderr` | Standard streams | `sys.stderr.write("Error\n")` | Read/write streams |
| `sys.path` | Module search paths | `sys.path.insert(0, '/my/lib')` | Add to import path |
| `sys.version` | Python version string | `sys.version` | Check Python version |
| `sys.version_info` | Python version tuple | `sys.version_info >= (3, 11)` | Version comparison |
| `sys.platform` | OS platform string | `sys.platform` → `'darwin'` | OS detection |
| `sys.getsizeof(obj)` | Memory size of object in bytes | `sys.getsizeof([1,2,3])` | Memory inspection |
| `sys.maxsize` | Largest int on this platform | `sys.maxsize` | Platform-aware int limit |
| `sys.modules` | Dict of loaded modules | `'pandas' in sys.modules` | Check if module loaded |
| `sys.executable` | Path to current Python interpreter | `sys.executable` | Know which Python/venv is active |
| `sys.stdin.isatty()` | True if stdin is a terminal (not piped data) | `sys.stdin.isatty()` | Detect piped input vs interactive |

```python
import sys

# Read CLI args (sys.argv[0] is the script name)
if len(sys.argv) < 2:
    print("Usage: script.py <input_file>", file=sys.stderr)
    sys.exit(1)
input_file = sys.argv[1]

# Version guard
if sys.version_info < (3, 10):
    raise RuntimeError("Python 3.10+ required")

# Write to stderr (errors/logs) vs stdout (data output)
print("Error: file not found", file=sys.stderr)
print(result_data)                            # Goes to stdout

# Add local lib to import path
sys.path.insert(0, 'src/')
import my_module

# Read from stdin — useful for piped input
# Usage: cat data.csv | python script.py
if not sys.stdin.isatty():
    for line in sys.stdin:
        process(line.strip())

# Or read all piped input at once
data = sys.stdin.read()

# Know which Python/venv is running (useful in multi-env setups)
print(sys.executable)   # e.g. /Users/user/.venv/bin/python
```

---

## types

Utility types and type-checking helpers for dynamic code.

```python
import types
```

| Class / Function | Description | Example | Use Case |
|------------------|-------------|---------|----------|
| `types.SimpleNamespace(**kwargs)` | Simple object with attributes | `ns = SimpleNamespace(x=1, y=2)` | Ad-hoc config object |
| `types.MappingProxyType(dict)` | Read-only view of a dict | `MappingProxyType({'a': 1})` | Immutable dict |
| `types.FunctionType` | Type of a function | `isinstance(f, types.FunctionType)` | Check if callable is a function |
| `types.LambdaType` | Type of a lambda (same as FunctionType) | `isinstance(fn, types.LambdaType)` | Lambda detection |
| `types.ModuleType` | Type of a module | `isinstance(m, types.ModuleType)` | Check if object is a module |
| `types.NoneType` | Type of `None` (3.10+) | `isinstance(x, types.NoneType)` | Explicit None type check |
| `types.UnionType` | Type of `X \| Y` union (3.10+) | `int \| str` is `types.UnionType` | Introspect union types |

```python
from types import SimpleNamespace, MappingProxyType

# SimpleNamespace — quick config/settings object
config = SimpleNamespace(
    host='localhost',
    port=5432,
    db='loans',
)
print(config.host)  # 'localhost'
config.host = 'prod-db'  # Mutable

# MappingProxyType — expose a dict as read-only
_DEFAULTS = {'timeout': 30, 'retries': 3}
DEFAULTS = MappingProxyType(_DEFAULTS)
DEFAULTS['timeout']  # 30
DEFAULTS['timeout'] = 60  # TypeError: read-only
```

---

## decimal

Precise decimal arithmetic — critical for financial data. Floats are binary fractions and silently lose precision.

```python
from decimal import Decimal, getcontext, ROUND_HALF_UP, ROUND_DOWN, ROUND_CEILING
```

| Class / Function | Description | Example | Use Case |
|------------------|-------------|---------|----------|
| `Decimal('value')` | Create exact decimal **from string** | `Decimal('15000.50')` | Always use a string — never a float |
| `Decimal(int)` | Create exact decimal from integer | `Decimal(15000)` | Safe — integers are exact |
| `getcontext().prec` | Get/set global decimal precision | `getcontext().prec = 28` | Control significant digits |
| `localcontext()` | Temporary scoped precision context | `with localcontext() as ctx:` | Change precision for one block |
| `.quantize(exp, rounding)` | Round to a specific decimal place | `amount.quantize(Decimal('0.01'), ROUND_HALF_UP)` | Round to cents |
| `ROUND_HALF_UP` | Standard rounding (0.5 rounds up) | N/A | Financial rounding |
| `ROUND_DOWN` | Truncate toward zero | N/A | Fee truncation |
| `ROUND_CEILING` | Always round up | N/A | Round up to next cent |
| `.is_nan()` / `.is_infinite()` | Check for special values | `Decimal('NaN').is_nan()` | Validate before arithmetic |

```python
from decimal import Decimal, getcontext, ROUND_HALF_UP

# ⚠️  Float precision problem
0.1 + 0.2                              # 0.30000000000000004  ← wrong
Decimal('0.1') + Decimal('0.2')        # Decimal('0.3')       ← correct

# Always construct from STRING, not float
Decimal(0.1)    # Decimal('0.1000000000000000055511151231257827021181583404541015625') ← wrong
Decimal('0.1')  # Decimal('0.1')  ← correct

# Round to 2 decimal places (cents)
amount = Decimal('15000.567')
rounded = amount.quantize(Decimal('0.01'), rounding=ROUND_HALF_UP)
# Decimal('15000.57')

# Percentage calculation
rate = Decimal('0.0325')
interest = (Decimal('50000') * rate).quantize(Decimal('0.01'), ROUND_HALF_UP)
# Decimal('1625.00')

# Convert back to float only after all calculations are done
float(rounded)   # 15000.57
```

> **Tip:** Store monetary values as integers (cents) or `Decimal`-compatible strings in your database — never as `float`.

---

## io

In-memory file-like objects — pass text or bytes to anything that expects a file without touching disk.

```python
from io import StringIO, BytesIO
```

| Class | Description | Example | Use Case |
|-------|-------------|---------|----------|
| `StringIO(text)` | In-memory text stream | `StringIO("a,b\n1,2\n")` | Pass CSV/JSON string to a reader |
| `BytesIO(data)` | In-memory binary stream | `BytesIO(b'\x89PNG...')` | Pass bytes to a binary reader |
| `.read()` | Read all content | `buf.read()` | Read the whole buffer |
| `.write(data)` | Write to the buffer | `buf.write("line\n")` | Build content programmatically |
| `.getvalue()` | Return full buffer contents as str/bytes | `buf.getvalue()` | Get final result |
| `.seek(0)` | Move read cursor back to start | `buf.seek(0)` | Rewind before reading |
| `.tell()` | Current cursor position | `buf.tell()` | Check position |

```python
from io import StringIO, BytesIO
import csv, json, gzip

# Pass a CSV string directly to DictReader — no temp file needed
raw_csv = "id,amount,status\n1,1000,open\n2,2500,closed\n"
reader = csv.DictReader(StringIO(raw_csv))
rows = list(reader)

# Write CSV to a string buffer (e.g., to send as an API response body)
buf = StringIO()
writer = csv.DictWriter(buf, fieldnames=['id', 'amount'])
writer.writeheader()
writer.writerows([{'id': 1, 'amount': 1000}])
csv_string = buf.getvalue()

# Useful in tests — mock a file without touching the filesystem
def test_parse_csv():
    fake_file = StringIO("name,value\nfoo,1\nbar,2\n")
    result = parse_csv(fake_file)
    assert len(result) == 2

# BytesIO — decompress gzip bytes from an API or S3 without writing to disk
compressed = download_from_s3('data.csv.gz')   # returns bytes
with gzip.open(BytesIO(compressed)) as f:
    content = f.read().decode('utf-8')
reader = csv.DictReader(StringIO(content))
```

---

## subprocess

Run shell commands and external programs from Python.

```python
import subprocess
```

| Function / Arg | Description | Key Args | Example | Use Case |
|----------------|-------------|----------|---------|----------|
| `subprocess.run(cmd)` | Run a command and wait for it to finish | `capture_output`, `text`, `check`, `cwd`, `env` | `subprocess.run(['dbt', 'run'])` | Run a command |
| `capture_output=True` | Capture stdout and stderr | N/A | `run(cmd, capture_output=True, text=True)` | Read command output |
| `text=True` | Decode stdout/stderr as strings (not bytes) | N/A | `run(cmd, text=True)` | Get string output |
| `check=True` | Raise `CalledProcessError` if exit code ≠ 0 | N/A | `run(cmd, check=True)` | Fail fast on error |
| `cwd='path'` | Set working directory for the command | N/A | `run(cmd, cwd='/project')` | Run in a specific directory |
| `env=dict` | Set environment variables for the command | N/A | `run(cmd, env={**os.environ, 'KEY': 'val'})` | Inject env vars |
| `result.stdout` | Command's stdout output | N/A | `result.stdout` | Read output |
| `result.returncode` | Exit code (0 = success) | N/A | `result.returncode` | Check success |
| `subprocess.CalledProcessError` | Raised when `check=True` and exit ≠ 0 | N/A | `except subprocess.CalledProcessError as e:` | Handle failures |

```python
import subprocess, os

# Run dbt and capture output
result = subprocess.run(
    ['dbt', 'run', '--select', 'my_model'],
    capture_output=True,
    text=True,
    cwd='/project',
)
if result.returncode != 0:
    print(result.stderr)
    raise RuntimeError("dbt run failed")
print(result.stdout)

# Raise automatically on non-zero exit
try:
    subprocess.run(['python', 'validate.py'], check=True, capture_output=True, text=True)
except subprocess.CalledProcessError as e:
    print(f"Failed (exit {e.returncode}):\n{e.stderr}")
    raise

# Pass extra environment variables (inherit current env + override/add)
env = {**os.environ, 'DBT_TARGET': 'prod', 'SNOWFLAKE_ROLE': 'transformer'}
subprocess.run(['dbt', 'run'], env=env, check=True)

# Capture output and parse it
result = subprocess.run(['git', 'rev-parse', 'HEAD'], capture_output=True, text=True)
commit_hash = result.stdout.strip()
```

> **Tip:** Avoid `shell=True` — it passes the command to a shell as a string, which is a security risk (command injection). Use a list of arguments instead.

---

## functools

Higher-order functions: caching, partial application, and function composition.

```python
from functools import lru_cache, cache, partial, reduce, wraps
```

| Function | Description | Key Args | Example | Use Case |
|----------|-------------|----------|---------|----------|
| `@lru_cache(maxsize=n)` | Cache return values, evict oldest when full | `maxsize` (None = unlimited) | `@lru_cache(maxsize=256)` | Cache expensive repeated lookups |
| `@cache` | Unbounded cache — shorthand for `lru_cache(maxsize=None)` (3.9+) | N/A | `@cache` | Cache with no eviction |
| `.cache_clear()` | Clear a cached function's cache | N/A | `get_rate.cache_clear()` | Reset stale cached values |
| `.cache_info()` | Show hits, misses, current size | N/A | `get_rate.cache_info()` | Debug cache efficiency |
| `partial(func, *args, **kwargs)` | Pre-fill some arguments to create a new function | N/A | `partial(round, ndigits=2)` | Fix some args, vary others |
| `reduce(func, iterable, initial)` | Fold iterable down to a single value | `initial` | `reduce(lambda a, b: a + b, [1,2,3], 0)` | Aggregate a sequence |
| `@wraps(func)` | Preserve `__name__`, `__doc__` when decorating | N/A | `@wraps(func)` | Use inside custom decorators |

```python
from functools import lru_cache, cache, partial, reduce, wraps

# lru_cache — cache expensive lookups (e.g., DB queries, API calls)
@lru_cache(maxsize=512)
def get_exchange_rate(currency: str, date: str) -> float:
    return fetch_from_api(currency, date)   # Only called once per unique (currency, date)

get_exchange_rate('USD', '2024-01-15')  # API call
get_exchange_rate('USD', '2024-01-15')  # Cache hit — no API call
get_exchange_rate.cache_info()          # CacheInfo(hits=1, misses=1, ...)
get_exchange_rate.cache_clear()         # Invalidate when data changes

# partial — create specialized versions of a function
round2 = partial(round, ndigits=2)
round2(15000.5678)   # 15000.57

import csv
tsv_reader = partial(csv.reader, delimiter='\t')
# tsv_reader(file) is now equivalent to csv.reader(file, delimiter='\t')

# reduce — aggregate over a list
total = reduce(lambda acc, row: acc + float(row['amount']), rows, 0.0)

# wraps — preserve function metadata inside decorators
def retry(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        for attempt in range(3):
            try:
                return func(*args, **kwargs)
            except Exception:
                if attempt == 2:
                    raise
    return wrapper
```

---

## enum

Define named constants — cleaner than bare strings or magic numbers.

```python
from enum import Enum, auto, IntEnum
```

| Class / Feature | Description | Example | Use Case |
|----------------|-------------|---------|----------|
| `class MyEnum(Enum)` | Define an enumeration | `class Status(Enum):` | Named constants |
| `auto()` | Auto-assign integer values | `OPEN = auto()` | When the actual value doesn't matter |
| `MyEnum.VALUE` | Access a member | `Status.OPEN` | Use a constant |
| `MyEnum(value)` | Look up member by value | `Status('open')` | Parse string/int from DB or API |
| `MyEnum['NAME']` | Look up member by name string | `Status['OPEN']` | Access by name |
| `.value` | Get the underlying value | `Status.OPEN.value` | Extract raw value |
| `.name` | Get the member name as string | `Status.OPEN.name` | Get string key |
| `for m in MyEnum` | Iterate all members | `for s in Status:` | List all valid values |
| `class MyEnum(str, Enum)` | String enum — members behave as plain strings | `class Status(str, Enum):` | JSON-serializable, no `.value` needed |

```python
from enum import Enum, auto

class PipelineStatus(Enum):
    PENDING = 'pending'
    RUNNING = 'running'
    SUCCESS = 'success'
    FAILED  = 'failed'

# Use instead of raw strings
status = PipelineStatus.RUNNING
if status == PipelineStatus.FAILED:
    send_alert()

# Parse from data (e.g., a database value)
db_value = 'success'
status = PipelineStatus(db_value)   # PipelineStatus.SUCCESS

# List all valid values (useful for validation or documentation)
valid = [s.value for s in PipelineStatus]
# ['pending', 'running', 'success', 'failed']

# str Enum — members ARE strings, so they serialize to JSON directly
class DataQuality(str, Enum):
    PASS_ = 'pass'
    WARN  = 'warn'
    FAIL  = 'fail'

import json
json.dumps({'quality': DataQuality.PASS_})  # '{"quality": "pass"}' ← works
```

---

## statistics

Descriptive statistics without numpy or pandas.

```python
import statistics
```

| Function | Description | Example | Use Case |
|----------|-------------|---------|----------|
| `statistics.mean(data)` | Arithmetic mean | `statistics.mean([1, 2, 3, 4])` | Average value |
| `statistics.median(data)` | Middle value | `statistics.median([1, 2, 3, 100])` | Robust central tendency |
| `statistics.mode(data)` | Most common value | `statistics.mode(['a','b','a'])` | Most frequent category |
| `statistics.multimode(data)` | All most common values (handles ties) | `statistics.multimode([1,1,2,2])` | Tied modes |
| `statistics.stdev(data)` | Sample standard deviation | `statistics.stdev(values)` | Spread of a sample |
| `statistics.pstdev(data)` | Population standard deviation | `statistics.pstdev(values)` | Spread of full population |
| `statistics.variance(data)` | Sample variance | `statistics.variance(values)` | Variance of a sample |
| `statistics.quantiles(data, n=4)` | Divide data into n equal groups | `statistics.quantiles(values, n=4)` | Quartiles (n=4), deciles (n=10) |
| `statistics.fmean(data)` | Fast float mean (3.8+) | `statistics.fmean(values)` | Faster than mean() for floats |
| `statistics.correlation(x, y)` | Pearson correlation coefficient (3.10+) | `statistics.correlation(x, y)` | Linear relationship between two series |
| `statistics.linear_regression(x, y)` | Slope and intercept (3.10+) | `statistics.linear_regression(x, y)` | Trend line |

```python
import statistics

balances = [5000, 12000, 8500, 150000, 7200, 9800, 11000]

statistics.mean(balances)      # 29071.4 — pulled up by the outlier
statistics.median(balances)    # 9800    — less affected by the 150k outlier
statistics.stdev(balances)     # spread around the mean

# Quartiles
q1, q2, q3 = statistics.quantiles(balances, n=4)

# Quick data quality check — flag outliers beyond 3 standard deviations
mean = statistics.mean(balances)
std  = statistics.stdev(balances)
outliers = [x for x in balances if abs(x - mean) > 3 * std]

# Correlation between two numeric series (Python 3.10+)
days_open        = [1,  5,  10,  15,  30]
balance_at_close = [100, 500, 1200, 1800, 4000]
statistics.correlation(days_open, balance_at_close)  # e.g. 0.998
```

---

## time

Measure elapsed time, add delays, and generate timestamps.

```python
import time
```

| Function | Description | Example | Use Case |
|----------|-------------|---------|----------|
| `time.time()` | Current Unix timestamp as float seconds | `time.time()` | Timestamp an event |
| `time.perf_counter()` | High-resolution elapsed timer | `time.perf_counter()` | Benchmark code accurately |
| `time.monotonic()` | Monotonic clock — never goes backward | `time.monotonic()` | Duration measurement (safe across sleeps) |
| `time.sleep(seconds)` | Pause execution | `time.sleep(1.5)` | Retry backoff, rate limiting |
| `time.strftime(fmt)` | Format current local time as string | `time.strftime('%Y%m%d_%H%M%S')` | Timestamped filenames |
| `time.gmtime(ts)` | Convert Unix timestamp to UTC struct | `time.gmtime()` | UTC time struct |
| `time.localtime(ts)` | Convert Unix timestamp to local struct | `time.localtime()` | Local time struct |

```python
import time

# Time a pipeline step
start = time.perf_counter()
process_file('large_data.csv')
elapsed = time.perf_counter() - start
print(f"Processed in {elapsed:.2f}s")

# Timestamped output filename
filename = f"export_{time.strftime('%Y%m%d_%H%M%S')}.csv"
# → export_20240115_093000.csv

# Retry with exponential backoff
def call_with_retry(fn, retries=3, base_delay=1.0):
    for attempt in range(retries):
        try:
            return fn()
        except Exception as e:
            if attempt == retries - 1:
                raise
            delay = base_delay * (2 ** attempt)   # 1s, 2s, 4s
            print(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay}s...")
            time.sleep(delay)

# Rate limiting — stay under API request limits
for item in items:
    process(item)
    time.sleep(0.1)   # Max ~10 requests/second
```

---

## math

Standard mathematical functions and constants.

```python
import math
```

| Function / Constant | Description | Example | Use Case |
|--------------------|-------------|---------|----------|
| `math.floor(x)` | Round down to nearest integer | `math.floor(3.9)` → `3` | Floor division |
| `math.ceil(x)` | Round up to nearest integer | `math.ceil(3.1)` → `4` | Ceiling — always round up |
| `math.trunc(x)` | Truncate toward zero | `math.trunc(-3.9)` → `-3` | Remove decimal part |
| `math.sqrt(x)` | Square root | `math.sqrt(16)` → `4.0` | Root calculations |
| `math.log(x, base)` | Logarithm (default: natural log) | `math.log(100, 10)` → `2.0` | Log scaling |
| `math.log2(x)` | Base-2 logarithm | `math.log2(1024)` → `10.0` | Bit-level calculations |
| `math.log10(x)` | Base-10 logarithm | `math.log10(1000)` → `3.0` | Order of magnitude |
| `math.pow(x, y)` | x to the power y (returns float) | `math.pow(2, 10)` → `1024.0` | Exponentiation |
| `math.exp(x)` | e raised to power x | `math.exp(1)` → `2.718...` | Exponential growth |
| `math.factorial(n)` | n! | `math.factorial(5)` → `120` | Combinatorics |
| `math.comb(n, k)` | Combinations n choose k (3.8+) | `math.comb(10, 3)` → `120` | Unique combinations |
| `math.gcd(a, b)` | Greatest common divisor | `math.gcd(12, 8)` → `4` | Simplify fractions |
| `math.isnan(x)` | True if x is NaN | `math.isnan(float('nan'))` → `True` | Detect missing floats |
| `math.isinf(x)` | True if x is infinite | `math.isinf(float('inf'))` → `True` | Detect overflow |
| `math.isfinite(x)` | True if x is neither NaN nor infinite | `math.isfinite(3.14)` → `True` | Validate float values |
| `math.isclose(a, b)` | Float equality with tolerance | `math.isclose(0.1+0.2, 0.3)` → `True` | Safe float comparison |
| `math.pi` | π ≈ 3.14159... | `math.pi` | Circle calculations |
| `math.e` | Euler's number ≈ 2.71828... | `math.e` | Natural exponent |
| `math.inf` | Positive infinity | `math.inf` | Sentinel max value |
| `math.nan` | Not a Number | `math.nan` | Sentinel missing float |

```python
import math

# Safe float comparison (avoids 0.1 + 0.2 != 0.3 issue)
math.isclose(0.1 + 0.2, 0.3)                    # True
math.isclose(1000.01, 1000.02, rel_tol=1e-3)    # True (within 0.1%)

# Detect NaN / inf in float columns before processing
values = [1.0, float('nan'), 3.0, float('inf')]
clean = [v for v in values if math.isfinite(v)]  # [1.0, 3.0]

# Batch size — use ceil to ensure the last partial batch is included
total_rows = 10_003
batch_size = 1_000
num_batches = math.ceil(total_rows / batch_size)   # 11

# Log scaling — compress skewed distributions for analysis
import statistics
log_balances = [math.log10(b) for b in balances if b > 0]
statistics.mean(log_balances)

# math.inf as a sentinel "worst case" starting value
min_balance = math.inf
for row in rows:
    if row['balance'] < min_balance:
        min_balance = row['balance']
```

