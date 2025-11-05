# Python Automation Test Patterns

## Infrastructure
- **Test framework:** pytest
- **Test runner:** `pytest` or `python -m pytest`
- **Config file:** `pytest.ini` or `pyproject.toml`
- **Coverage:** pytest-cov

## Setup Patterns

### pytest.ini
```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --cov=src --cov-report=html
```

### tests/conftest.py
```python
import pytest
import tempfile
import os

@pytest.fixture
def temp_dir():
    """Provide temporary directory for tests."""
    with tempfile.TemporaryDirectory() as tmpdir:
        yield tmpdir

@pytest.fixture
def mock_env(monkeypatch):
    """Mock environment variables."""
    monkeypatch.setenv('API_KEY', 'test-key')
    monkeypatch.setenv('DEBUG', 'true')
```

## Code Patterns

### Function Testing
```python
import pytest
from src.utils import parse_csv, validate_email

def test_parse_csv():
    data = "name,age\nJohn,30\nJane,25"
    result = parse_csv(data)

    assert len(result) == 2
    assert result[0]['name'] == 'John'
    assert result[0]['age'] == '30'

def test_validate_email():
    assert validate_email('user@example.com') == True
    assert validate_email('invalid-email') == False

@pytest.mark.parametrize('email,expected', [
    ('user@example.com', True),
    ('test@test.co.uk', True),
    ('invalid', False),
    ('@example.com', False),
])
def test_validate_email_parametrized(email, expected):
    assert validate_email(email) == expected
```

### File Operations Testing
```python
def test_read_config(temp_dir):
    config_path = os.path.join(temp_dir, 'config.json')

    with open(config_path, 'w') as f:
        f.write('{"key": "value"}')

    result = read_config(config_path)
    assert result['key'] == 'value'
```

### API Mocking
```python
from unittest.mock import Mock, patch
import requests

@patch('requests.get')
def test_fetch_data(mock_get):
    mock_response = Mock()
    mock_response.json.return_value = {'data': 'test'}
    mock_response.status_code = 200
    mock_get.return_value = mock_response

    result = fetch_data('http://api.example.com')
    assert result['data'] == 'test'
```

### Exception Testing
```python
def test_division_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

def test_invalid_input():
    with pytest.raises(ValueError, match='Invalid format'):
        parse_date('not-a-date')
```

### Class Testing
```python
class TestDataProcessor:
    @pytest.fixture(autouse=True)
    def setup(self):
        self.processor = DataProcessor()
        yield
        self.processor.cleanup()

    def test_process_data(self):
        result = self.processor.process([1, 2, 3])
        assert len(result) == 3
```

## Learned Patterns
(TDD skill will append learned patterns here)
