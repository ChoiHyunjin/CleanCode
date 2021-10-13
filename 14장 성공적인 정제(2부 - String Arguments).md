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