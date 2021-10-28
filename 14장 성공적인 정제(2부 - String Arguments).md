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

추상 메소드 set을 만들었다. 그리고 다음과 같이 BooleanArgumentMarshaller에 구현했다.

```jsx
private

class BooleanArgumentMarshaler extends ArgumentMarshaler {
  public void

  set(String

  s
) {
  booleanValue = true;
}
}

// use
private
void setBooleanArg(char
argChar, boolean
value
)
{
  booleanArgs.get(argChar).set("true");
}
```

set 을 옮겼으니, ArgumentMarshaler.setBoolean 메서드는 제거한다.

참고로 추상 set 함수는 String 인수를 받아들이나, BooleanArgumentMarshaler는 인수를 사용하지 않는다. *(이게 뭔말임?)* 인수는
STring/IntegerArgumentMarshaler에서 필요하기에 정의했다.

다음으로 get 메서드를 옮겼다. 반환 type은 Boolean이다.

```java
public boolean getBoolean(char arg){
    Args.ArgumentMarshaler am=booleanArgs.get(arg);
    return am!=null&&(Boolean)am.get();
    }

// 위 코드를 컴파일할 목적으로 get 추가
private abstract class ArgumentMarshaler { 
	...

  public Object get() {
    return null;
  }
}
```

위 코드를 컴파일할 목적으로 get 추가했다. *(????)*

```java
private abstract class ArgumentMarshaler {
  protected boolean booleanValue = false; 
	...

  public abstract Object get();
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
  public void set(String s) {
    booleanValue = true;
  }

  public Object get() {
    return booleanValue;
  }
}
```

코드는 컴파일됐지만, 테스트는 실패했다. 테스트 통과를 위해 get을 추상 메서드로 만들고 BoolArgumentMarshaler에 get을 추가했다.

```java
  private void setStringArg(char argChar)throws ArgsException{
    currentArgument++;
    try{
    stringArgs.get(argChar).set(args[currentArgument]);
    }catch(ArrayIndexOutOfBoundsException e){
    valid=false;
    errorArgumentId=argChar;
    errorCode=ErrorCode.MISSING_STRING;
    throw new ArgsException();
    }
    }
    ...
public String getString(char arg){
    Args.ArgumentMarshaler am=stringArgs.get(arg);
    return am==null?"":(String)am.get();
    }
    ...

private abstract class ArgumentMarshaler {
  private int integerValue;

  public void setInteger(int i) {
    integerValue = i;
  }

  public int getInteger() {
    return integerValue;
  }

  public abstract void set(String s);

  public abstract Object get();
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
  private boolean booleanValue = false;

  public void set(String s) {
    booleanValue = true;
  }

  public Object get() {
    return booleanValue;
  }
}

private class StringArgumentMarshaler extends ArgumentMarshaler {
  private String stringValue = "";

  public void set(String s) {
    stringValue = s;
  }

  public Object get() {
    return stringValue;
  }
}

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
  public void set(String s) {
  }

  public Object get() {
    return null;
  }
}
}
```
