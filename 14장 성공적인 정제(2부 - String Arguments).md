# 14장 성공적인 정제(2부 - String Arguments)

## Questions

1.

# Organize

## String Arguments

boolean 처리와 유사하다.

- Hashmap으로 변환
- ArgumentMarshaler 클래스에 마샬링 기능 구현
- parse, set, get 함수 추가

```jsx
private Map<Character, **ArgumentMarshaler**> stringArgs = new HashMap<Character, **ArgumentMarshaler**>();
...
private void parseStringSchemaElement(char elementId) {
  stringArgs.put(elementId, **new StringArgumentMarshaler()**);
} ...
private void setStringArg(char argChar) throws ArgsException {
  currentArgument++;
  try {
    stringArgs.**get**(argChar).**setString**(args[currentArgument]);
  } catch (ArrayIndexOutOfBoundsException e) {
    valid = false;
    errorArgumentId = argChar;
    errorCode = ErrorCode.MISSING_STRING;
    throw new ArgsException();
  }
}
...
public String getString(char arg) {
**Args.ArgumentMarshaler** am = stringArgs.get(arg);
  return **am == null ? ""** : am.getString();
}
...
private class ArgumentMarshaler {
  private boolean booleanValue = false;
  **private String stringValue;**

  public void setBoolean(boolean value) {
  booleanValue = value;
}
public boolean getBoolean() {
  return booleanValue;
}
**public void setString(String s) {
  stringValue = s;
}
public String getString() {
  return stringValue == null ? "" : stringValue;
}**
}
```

당연히, 이 변화들은 기존의 테스트가 통과하도록 수정되었다.

다음은 int Marshaler를 추가했다

```jsx
private Map<Character, **ArgumentMarshaler**> intArgs = 
	new HashMap<Character, **ArgumentMarshaler**>();
...
private void parseIntegerSchemaElement(char elementId) { 
	intArgs.put(elementId, new **IntegerArgumentMarshaler**());
} 
...
private void setIntArg(char argChar) throws ArgsException { 
	currentArgument++;
	String parameter = null;
	try {
		parameter = args[currentArgument];
		intArgs.**get**(argChar).**setInteger**(Integer.parseInt(parameter));
	} catch (ArrayIndexOutOfBoundsException e) {
		valid = false;
		errorArgumentId = argChar;
		errorCode = ErrorCode.MISSING_INTEGER; 
		throw new ArgsException();
	} catch (NumberFormatException e) { 
		valid = false;
		errorArgumentId = argChar; errorParameter = parameter;
		errorCode = ErrorCode.INVALID_INTEGER; throw new ArgsException();
	} 
}
...
public int getInt(char arg) {
	**Args.ArgumentMarshaler am** = intArgs.get(arg);
	return **am == null ? 0** : am.getInteger(); 
}
...
private class ArgumentMarshaler {
	private boolean booleanValue = false; 
	private String stringValue;
	**private int integerValue;**
	public void setBoolean(boolean value) { 
		booleanValue = value;
	}
	public boolean getBoolean() { 
		return booleanValue;
	}
	public void setString(String s) { 
		stringValue = s;
	}
	public String getString() {
		return stringValue == null ? "" : stringValue;
	}
	**public void setInteger(int i) { 
		integerValue = i;
	}
	public int getInteger() { 
		return integerValue;
	}**
}
```

모든 먀셜링은 ArguMentMarshaler로 이동했고, 이제 함수 요소들을 밀어넣어 보자. 첫째로는  *setBoolean*을 BooleanArgumentMarshaller에 추가하고, 올바른지 확인하자.

```jsx
private **abstract** class ArgumentMarshaler { 
	**protected** boolean booleanValue = false; private String stringValue;
	private int integerValue;
	public void setBoolean(boolean value) { 
		booleanValue = value;
	}
	public boolean getBoolean() {
		return booleanValue;
	}
	public void setString(String s) { 
		stringValue = s;
	}
	public String getString() {
		return stringValue == null ? "" : stringValue;
	}
	public void setInteger(int i) { 
		integerValue = i;
	}
	public int getInteger() { 
		return integerValue;
	}
	**public abstract void set(String s);**
}
```
