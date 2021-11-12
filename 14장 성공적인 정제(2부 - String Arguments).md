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

코드는 컴파일됐지만, 테스트는 실패했다. 테스트 통과를 위해 get을 추상 메서드로 만들고 BoolArgumentMarshaler에 get을 추가했다.

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

모든 테스트에 만족하는 상태에서, set/get을 BooleanArgumentMarshaler로 이동시켰다. ArgumentMarshaler의 getBoolean 함수를 제거하고, protecte 변수인 booleanValue는 BooleanArgumentMarshaler로 이동시키면서 private으로 선언한다.

String인 경우도 동일한 방식으로 변경한다. get/set을 옮긴 후 사용하지 않는 함수를 제거하고 변수를 옮겼다.

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
```

integer인 경우도 같은 과정을 반복한다. integer의 경우에는 parse에서 예외를 던질 수도 있으므로, 구현이 조금 더 복잡하다. 하지만 NumberFormatException이라는 개념이 IntegerArgumentMarshaler에 숨겨지므로 결과는 더 좋았다.

```java
private boolean isIntArg(char argChar){return intArgs.containsKey(argChar);}
private void setIntArg(char argChar)throws ArgsException{currentArgument++;
    String parameter=null;
    try{
    parameter=args[currentArgument];
    intArgs.get(argChar).set(parameter);
    }catch(ArrayIndexOutOfBoundsException e){
    valid=false;
    errorArgumentId=argChar;
    errorCode=ErrorCode.MISSING_INTEGER;throw new ArgsException();
    }catch(ArgsException e){
    valid=false;
    errorArgumentId=argChar;errorParameter=parameter;
    errorCode=ErrorCode.INVALID_INTEGER;throw e;
    }}
    ...
private void setBooleanArg(char argChar){
    try{
    booleanArgs.get(argChar).set("true");
    }catch(ArgsException e){
    }
    }...
public int getInt(char arg){
	Args.ArgumentMarshaler am=intArgs.get(arg);
	return am==null?0:(Integer)am.get();
  }...

private abstract class ArgumentMarshaler {
  public abstract void set(String s) throws ArgsException;

  public abstract Object get();
} ...

private class IntegerArgumentMarshaler extends ArgumentMarshaler {
  private int intValue = 0;

  public void set(String s) throws ArgsException {
    try {
      intValue = Integer.parseInt(s);
    } catch (NumberFormatException e) {
      throw new ArgsException();
    }
  }

  public Object get() {
    return intValue;
  }
}
```

테스트 코드는 모두 통과했다.

## 인수 유형마다 만든 map 세 개를 줄이기

```jsx
public class Args {
...
    private Map<Character, ArgumentMarshaler> booleanArgs = 
    new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> stringArgs = 
    new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> intArgs = 
    new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> marshalers = 
    new HashMap<Character, ArgumentMarshaler>(); 
...
 private void parseBooleanSchemaElement(char elementId) {
    ArgumentMarshaler m = new BooleanArgumentMarshaler();
    booleanArgs.put(elementId, m);
    marshalers.put(elementId, m);
 }
 private void parseIntegerSchemaElement(char elementId) {
     ArgumentMarshaler m = new IntegerArgumentMarshaler();
     intArgs.put(elementId, m);
     marshalers.put(elementId, m);
 }
 private void parseStringSchemaElement(char elementId) {
     ArgumentMarshaler m = new StringArgumentMarshaler();
     stringArgs.put(elementId, m);
     marshalers.put(elementId, m);
 }
```

1. 우선 is***Arg를 다음과 같이 변경했다.

    ```jsx
    // before
    private boolean isBooleanArg(char argChar) {
    	return booleanArgs.containsKey(argChar);
    }
    
    // after
    private boolean isBooleanArg(char argChar) {
    	ArgumentMarshaler m = marshalers.get(argChar);
    	return m instanceof BooleanArgumentMarshaler;
    }
    // int & string, too
    private boolean isIntArg(char argChar) {
    	ArgumentMarshaler m = marshalers.get(argChar);
    	return m instanceof IntegerArgumentMarshaler;
    }
    private boolean isStringArg(char argChar) {
    	ArgumentMarshaler m = marshalers.get(argChar);
    	return m instanceof StringArgumentMarshaler;
    }
    ```

2. 반복되는 marshalers.get 또한 줄인다.

    ```jsx
    private boolean setArgument(char argChar) throws ArgsException {
    	ArgumentMarshaler m = marshalers.get(argChar);
    	if (isBooleanArg(m))
    		setBooleanArg(argChar);
    	else if (isStringArg(m))
    		setStringArg(argChar);
    	else if (isIntArg(m))
    		setIntArg(argChar);
    	else
    		return false;
    	return true;
    }
    private boolean isIntArg(ArgumentMarshaler m) {
    	return m instanceof IntegerArgumentMarshaler;
    }
    private boolean isStringArg(ArgumentMarshaler m) {
    	return m instanceof StringArgumentMarshaler;
    }
    private boolean isBooleanArg(ArgumentMarshaler m) {
    	return m instanceof BooleanArgumentMarshaler;
    }
    ```

3. is***Arg 메소드들도 하나로 합칠수 있게 되었다.

    ```jsx
    private boolean setArgument(char argChar) throws ArgsException {
    	ArgumentMarshaler m = marshalers.get(argChar);
    	if (m instanceof BooleanArgumentMarshaler)
    		setBooleanArg(argChar);
    	else if (m instanceof StringArgumentMarshaler)
    		setStringArg(argChar);
    	else if (m instanceof IntegerArgumentMarshaler)
    		setIntArg(argChar);
    	else
    		return false;
    	return true;
    }
    ```

4. set 함수의 인자들도 바꾼다. 그리고 setArgument함수 내부에 try-catch 로직도 추가한다.

    ```jsx
    private boolean setArgument(char argChar) throws ArgsException {
    	ArgumentMarshaler m = marshalers.get(argChar);
    	try {
    		if (m instanceof BooleanArgumentMarshaler)
    		setBooleanArg(m);
    		else if (m instanceof StringArgumentMarshaler)
    		setStringArg(m);
    		else if (m instanceof IntegerArgumentMarshaler)
    		setIntArg(m);
    		else
    		return false;
    	} catch (ArgsException e) {
    		valid = false;
    		errorArgumentId = argChar;
    		throw e;
    	}
    	return true;
    }
    private void setBooleanArg(ArgumentMarshaler m) {
    	try { 
    		m.set("true"); // was: booleanArgs.get(argChar).set("true");
    	} catch (ArgsException e) {
    	}
    }
    private void setIntArg(ArgumentMarshaler m) throws ArgsException {
    	currentArgument++;
    	String parameter = null;
    	try {
    		parameter = args[currentArgument]; m.set(parameter);
    	} catch (ArrayIndexOutOfBoundsException e) {
    		errorCode = ErrorCode.MISSING_INTEGER;
    		throw new ArgsException();
    	} catch (ArgsException e) {
    		errorParameter = parameter;
    		errorCode = ErrorCode.INVALID_INTEGER;
    		throw e;
    	}
    }
    private void setStringArg(ArgumentMarshaler m) throws ArgsException {
    	currentArgument++;
    	try { 
    		m.set(args[currentArgument]);
    	} catch (ArrayIndexOutOfBoundsException e) {
    		errorCode = ErrorCode.MISSING_STRING;
    		throw new ArgsException();
    	}
    }
    ```

5. getBoolean 함수에서 booleanArgs사용을 제거한다.

    ```jsx
    // before
    public boolean getBoolean(char arg) {
    	Args.ArgumentMarshaler am = booleanArgs.get(arg);
    	return am != null && (Boolean) am.get();
    }
    // after
    public boolean getBoolean(char arg) {
    	Args.ArgumentMarshaler am = marshalers.get(arg);
    	boolean b = false;
    	try {
    		b = am != null && (Boolean) am.get();
    	} catch (ClassCastException e) {
    		b = false;
    	}
    	return b;
    }
    ```

    - ClassCastException 사용 이유 : 인수 테스트(Acceptance Test) 케이스를 FitNess에서 구현했기 때문.
        - Fitness ?

          인수 테스트에 사용되는 도구

          [FitNesse를 활용한 인수테스트](https://wiblee.tistory.com/entry/FitNesse%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%9D%B8%EC%88%98%ED%85%8C%EC%8A%A4%ED%8A%B8)

6. booleanArgs 제거

    ```jsx
    private void parseBooleanSchemaElement(char elementId) {
    	ArgumentMarshaler m = new BooleanArgumentMarshaler();
    	~~booleanArgs.put(elementId, m);~~
    	marshalers.put(elementId, m);
    }
    ...
    public class Args {
    ...
    ~~private Map<Character, ArgumentMarshaler> booleanArgs = 
    	new HashMap<Character, ArgumentMarshaler>();~~
    private Map<Character, ArgumentMarshaler> stringArgs = 
    	new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> intArgs = 
    	new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> marshalers = 
    	new HashMap<Character, ArgumentMarshaler>(); 
    ...
    ```

7. String & Integer 또한 위처럼 변경하고, Map을 제거한다.

    ```java
    private void parseBooleanSchemaElement(char elementId) {
    	marshalers.put(elementId, new BooleanArgumentMarshaler());
    }
    private void parseIntegerSchemaElement(char elementId) {
    	marshalers.put(elementId, new IntegerArgumentMarshaler());
    }
    private void parseStringSchemaElement(char elementId) {
    	marshalers.put(elementId, new StringArgumentMarshaler());
    }
    ...
    public String getString(char arg) {
    	Args.ArgumentMarshaler am = marshalers.get(arg);
    	try {
    		return am == null ? "" : (String) am.get();
    	} catch (ClassCastException e) {
    		return "";
    	}
    }
    public int getInt(char arg) {
    	Args.ArgumentMarshaler am = marshalers.get(arg);
    	try {
    		return am == null ? 0 : (Integer) am.get();
    	} catch (Exception e) {
    		return 0;
    	}
    }
    ...
    public class Args {
    	...
    	~~private Map<Character, ArgumentMarshaler> stringArgs = 
    	new HashMap<Character, ArgumentMarshaler>();
    	private Map<Character, ArgumentMarshaler> intArgs = 
    	new HashMap<Character, ArgumentMarshaler>();~~
    	private Map<Character, ArgumentMarshaler> marshalers = 
    	new HashMap<Character, ArgumentMarshaler>(); 
    	...
    ```

8. parse 메서드 세 개를 인라인 코드로 변환했다.

    ```java
    private void parseSchemaElement(String element) throws ParseException {
    	...
    	if (isBooleanSchemaElement(elementTail))
    		marshalers.put(elementId, new BooleanArgumentMarshaler());
    	else if (isStringSchemaElement(elementTail))
    		marshalers.put(elementId, new StringArgumentMarshaler());
    	else if (isIntegerSchemaElement(elementTail)) {
    		marshalers.put(elementId, new IntegerArgumentMarshaler());
    	} else {
    		throw new ParseException(String.format(
    		"Argument: %c has invalid format: %s.", elementId, elementTail), 0);
    	}
    }
    ```


## 중간 점검

- 코드

    ```java
    package com.objectmentor.utilities.getopts;
        import java.text.ParseException;
        import java.util.*;
    
    public class Args {
      private String schema;
      private String[] args;
      private boolean valid = true;
      private Set<Character> unexpectedArguments = new TreeSet<Character>();
      private Map<Character, ArgumentMarshaler> marshalers =
          new HashMap<Character, ArgumentMarshaler>();
      private Set<Character> argsFound = new HashSet<Character>();
      private int currentArgument;
      private char errorArgumentId = '\0';
      private String errorParameter = "TILT";
      private ErrorCode errorCode = ErrorCode.OK;
    
      private enum ErrorCode {
        OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT
      }
    
      public Args(String schema, String[] args) throws ParseException {
        this.schema = schema;
        this.args = args;
        valid = parse();
      }
    
      private boolean parse() throws ParseException {
        if (schema.length() == 0 && args.length == 0)
          return true;
        parseSchema();
        try {
          parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
      }
    
      private boolean parseSchema() throws ParseException {
        for (String element : schema.split(",")) {
          if (element.length() > 0) {
            String trimmedElement = element.trim();
            parseSchemaElement(trimmedElement);
          }
        }
        return true;
      }
    
      private void parseSchemaElement(String element) throws ParseException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (isBooleanSchemaElement(elementTail))
          marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (isStringSchemaElement(elementTail))
          marshalers.put(elementId, new StringArgumentMarshaler());
        else if (isIntegerSchemaElement(elementTail)) {
          marshalers.put(elementId, new IntegerArgumentMarshaler());
        } else {
          throw new ParseException(String.format(
              "Argument: %c has invalid format: %s.", elementId, elementTail), 0);
        }
      }
    
      private void validateSchemaElementId(char elementId) throws ParseException {
        if (!Character.isLetter(elementId)) {
          throw new ParseException(
              "Bad character:" + elementId + "in Args format: " + schema, 0);
        }
      }
    
      private boolean isStringSchemaElement(String elementTail) {
        return elementTail.equals("*");
      }
    
      private boolean isBooleanSchemaElement(String elementTail) {
        return elementTail.length() == 0;
      }
    
      private boolean isIntegerSchemaElement(String elementTail) {
        return elementTail.equals("#");
      }
    
      private boolean parseArguments() throws ArgsException {
        for (currentArgument = 0; currentArgument < args.length; currentArgument++) {
          String arg = args[currentArgument];
          parseArgument(arg);
        }
        return true;
      }
    
      private void parseArgument(String arg) throws ArgsException {
        if (arg.startsWith("-"))
          parseElements(arg);
      }
    
      private void parseElements(String arg) throws ArgsException {
        for (int i = 1; i < arg.length(); i++)
          parseElement(arg.charAt(i));
      }
    
      private void parseElement(char argChar) throws ArgsException {
        if (setArgument(argChar))
          argsFound.add(argChar);
        else {
          unexpectedArguments.add(argChar);
          errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
          valid = false;
        }
      }
    
      private boolean setArgument(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        try {
          if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m);
          else if (m instanceof StringArgumentMarshaler)
            setStringArg(m);
          else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);
          else
            return false;
        } catch (ArgsException e) {
          valid = false;
          errorArgumentId = argChar;
          throw e;
        }
        return true;
      }
    
      private void setIntArg(ArgumentMarshaler m) throws ArgsException {
        currentArgument++;
        String parameter = null;
        try {
          parameter = args[currentArgument];
          m.set(parameter);
        } catch (ArrayIndexOutOfBoundsException e) {
          errorCode = ErrorCode.MISSING_INTEGER;
          throw new ArgsException();
        } catch (ArgsException e) {
          errorParameter = parameter;
          errorCode = ErrorCode.INVALID_INTEGER;
          throw e;
        }
      }
    
      private void setStringArg(ArgumentMarshaler m) throws ArgsException {
        currentArgument++;
        try {
          m.set(args[currentArgument]);
        } catch (ArrayIndexOutOfBoundsException e) {
          errorCode = ErrorCode.MISSING_STRING;
          throw new ArgsException();
        }
      }
    
      private void setBooleanArg(ArgumentMarshaler m) {
        try {
          m.set("true");
        } catch (ArgsException e) {
        }
      }
    
      public int cardinality() {
        return argsFound.size();
      }
    
      public String usage() {
        if (schema.length() > 0)
          return "-[" + schema + "]";
        else
          return "";
      }
    
      public String errorMessage() throws Exception {
        switch (errorCode) {
          case OK:
            throw new Exception("TILT: Should not get here.");
          case UNEXPECTED_ARGUMENT:
            return unexpectedArgumentMessage();
          case MISSING_STRING:
            return String.format("Could not find string parameter for -%c.",
                errorArgumentId);
          case INVALID_INTEGER:
            return String.format("Argument -%c expects an integer but was '%s'.",
                errorArgumentId, errorParameter);
          case MISSING_INTEGER:
            return String.format("Could not find integer parameter for -%c.",
                errorArgumentId);
        }
        return "";
      }
    
      private String unexpectedArgumentMessage() {
        StringBuffer message = new StringBuffer("Argument(s) -");
        for (char c : unexpectedArguments) {
          message.append(c);
        }
        message.append(" unexpected.");
        return message.toString();
      }
    
      public boolean getBoolean(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        boolean b = false;
        try {
          b = am != null && (Boolean) am.get();
        } catch (ClassCastException e) {
          b = false;
        }
        return b;
      }
    
      public String getString(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        try {
          return am == null ? "" : (String) am.get();
        } catch (ClassCastException e) {
          return "";
        }
      }
    
      public int getInt(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        try {
          return am == null ? 0 : (Integer) am.get();
        } catch (Exception e) {
          return 0;
        }
      }
    
      public boolean has(char arg) {
        return argsFound.contains(arg);
      }
    
      public boolean isValid() {
        return valid;
      }
    
      private class ArgsException extends Exception {
      }
    
      private abstract class ArgumentMarshaler {
        public abstract void set(String s) throws ArgsException;
    
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
        private int intValue = 0;
    
        public void set(String s) throws ArgsException {
          try {
            intValue = Integer.parseInt(s);
          } catch (NumberFormatException e) {
            throw new ArgsException();
          }
        }
    
        public Object get() {
          return intValue;
        }
      }
    }
    ```


개선점

- 첫 머리의 변수들
- setArgument의 유형을 일일이 확인하는 로직
- 모든 set 함수
- 오류 처리 코드

## setArgument의 유형 확인 로직 개선

- ArgumentMarshaler.set 호출로 변경
- 그러려면 set***Arg를 해당 ArgumentMarshaler 파생 클래스로 내려야함.
- But, setIntArg에서 인스턴스 변수 두 개를 씀.
  → args 배열을 list로 변환 후 Iterator를 set 함수로 전달

    ```java
    public class Args {
    	private String schema;
    	private String[] args;
    	private boolean valid = true;
    	private Set<Character> unexpectedArguments = new TreeSet<Character>();
    	private Map<Character, ArgumentMarshaler> marshalers = 
    	new HashMap<Character, ArgumentMarshaler>();
    	private Set<Character> argsFound = new HashSet<Character>();
    	private Iterator<String> currentArgument;
    	private char errorArgumentId = '\0';
    	private String errorParameter = "TILT";
    	private ErrorCode errorCode = ErrorCode.OK;
    	private List<String> argsList;
    	private enum ErrorCode {
    		OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT}
    		public Args(String schema, String[] args) throws ParseException {
    		this.schema = schema;
    		argsList = Arrays.asList(args);
    		valid = parse();
    	}
    	private boolean parse() throws ParseException {
    		if (schema.length() == 0 && argsList.size() == 0)
    			return true;
    		parseSchema();
    		try {
    			parseArguments();
    		} catch (ArgsException e) {
    		}
    		return valid;
    	}
    ---
    private boolean parseArguments() throws ArgsException {
    	for (currentArgument = argsList.iterator(); currentArgument.hasNext();) {
    		String arg = currentArgument.next(); parseArgument(arg);
    	}
    	return true;
    }
    ---
    private void setIntArg(ArgumentMarshaler m) throws ArgsException {
    	String parameter = null;
    	try {
    		parameter = currentArgument.next(); m.set(parameter);
    	} catch (NoSuchElementException e) {
    		errorCode = ErrorCode.MISSING_INTEGER;
    	throw new ArgsException();
    	} catch (ArgsException e) {
    		errorParameter = parameter;
    		errorCode = ErrorCode.INVALID_INTEGER;
    		throw e;
    	}
    }
    private void setStringArg(ArgumentMarshaler m) throws ArgsException {
    	try {
    		m.set(currentArgument.next());
    	} catch (NoSuchElementException e) {
    		errorCode = ErrorCode.MISSING_STRING;
    		throw new ArgsException();
    	}
    }
    ```

- set 함수를 적절한 파생 클래스로 내리기를 위한 선행작업으로 setArgument 수정

    ```java
    private boolean setArgument(char argChar) throws ArgsException { 
    	ArgumentMarshaler m = marshalers.get(argChar);
    	**if (m == null)
    		return false;** 
    	try {
    		if (m instanceof BooleanArgumentMarshaler) setBooleanArg(m);
    		else if (m instanceof StringArgumentMarshaler) setStringArg(m);
    		else if (m instanceof IntegerArgumentMarshaler) setIntArg(m);
    		~~else return false;~~
    	} catch (ArgsException e) { 
    		valid = false; 
    		errorArgumentId = argChar; 
    		throw e;
    	}
    	return true; 
    }
    ```

  이 수정은 if-else 체인을 완전히 제거하기 위해서 중요한 사항이다. 그래서 if-else 체인에서 오류 코드를 발생시켰다.

  이제 set 함수를 옮길 수 있다. 사소한 setBooleanArg 부터 준비한다. setBooleanArg함수가 BooleanArgumentMarshaler에 전달하는 것을 목표로 한다.

    ```java
    private boolean setArgument(char argChar) throws ArgsException { 
    	ArgumentMarshaler m = marshalers.get(argChar);
    	if (m == null)
    		return false; 
    	try {
    		if (m instanceof BooleanArgumentMarshaler) 
    			setBooleanArg(m, **currentArgument**);
    		else if (m instanceof StringArgumentMarshaler) setStringArg(m);
    		else if (m instanceof IntegerArgumentMarshaler) setIntArg(m);
    	} catch (ArgsException e) { 
    		valid = false; 
    		errorArgumentId = argChar; 
    		throw e;
    	}
    	return true; 
    }
    ---
    private void setBooleanArg(ArgumentMarshaler m, **Iterator<String> currentArgument**)
    throws ArgsException {
      ~~try {~~
    		m.set("true");
    	~~} catch (ArgsException e) { }~~
    }
    ```
