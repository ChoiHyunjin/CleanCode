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

  리팩토링 과정에서 try-catch가 제거됐다.

  setBooleanArg에서 Iterator를 사용하지 않음에도 추가한 이유는 ArgumentMarshaler의 추상 메서드로, setIntArg, setStringArg에서 필요하기 때문이다.

  setBooleanArg함수가 더이상 필요 없어졌다. ArgumentMarshaler에 set 함수를 추가하여 직접 호출하자.

    ```java
    private abstract class ArgumentMarshaler {
    	public abstract void set(Iterator<String> currentArgument) throws ArgsException;
    	public abstract void set(String s) throws ArgsException;
    	public abstract Object get(); 
    	}
    	
    	// ...
    	
    private class BooleanArgumentMarshaler extends ArgumentMarshaler { 
    	private boolean booleanValue = false;
    	public void set(Iterator<String> currentArgument) throws ArgsException { 
    		booleanValue = true;
    	}
    	public void set(String s) {
    		~~booleanValue = true;~~
    	}
    	public Object get() { 
    		return booleanValue;
    	} 
    }
    private class StringArgumentMarshaler extends ArgumentMarshaler { 
    	private String stringValue = "";
    	public void set(Iterator<String> currentArgument) throws ArgsException { }
    	public void set(String s) { 
    		stringValue = s;
    	}
    	public Object get() { 
    		return stringValue;
    	} 
    }
    private class IntegerArgumentMarshaler extends ArgumentMarshaler { 
    	private int intValue = 0;
    	public void set(Iterator<String> currentArgument) throws ArgsException { }
    	public void set(String s) throws ArgsException { 
    		try {
    			intValue = Integer.parseInt(s); 
    		} 
    		catch (NumberFormatException e) {
    			throw new ArgsException(); 
    		}
    	}
    	public Object get() { 
    		return intValue;
    	} 
    }
    ```

  이후 setBooleanArg를 제거

    ```java
    private boolean setArgument(char argChar) throws ArgsException {
      ArgumentMarshaler m = marshalers.get(argChar);
      if (m == null)
        return false;
      try {
        if (m instanceof BooleanArgumentMarshaler) m.set(currentArgument);
        else if (m instanceof StringArgumentMarshaler) setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler) setIntArg(m);
      } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
      }
      return true;
    }
    ```

  모든 테스트를 통과했다. set 함수는 BooleanArgumentMarshaler로 들어갔다.

    - String과 Integer 인수도 변경해준다.

        ```java
        private boolean setArgument(char argChar) throws ArgsException {
          ArgumentMarshaler m = marshalers.get(argChar);
          if (m == null)
            return false;
          try {
            if (m instanceof BooleanArgumentMarshaler) m.set(currentArgument);
            else if (m instanceof StringArgumentMarshaler)
              m.set(currentArgument);
            else if (m instanceof IntegerArgumentMarshaler)
              m.set(currentArgument);
          } catch (ArgsException e) {
            valid = false;
            errorArgumentId = argChar;
            throw e;
          }
          return true;
        }
        
        //---
        private class StringArgumentMarshaler extends ArgumentMarshaler {
          private String stringValue = "";
        
          public void set(Iterator<String> currentArgument) throws ArgsException {
            try {
              stringValue = currentArgument.next();
            } catch (NoSuchElementException e) {
              errorCode = ErrorCode.MISSING_STRING;
              throw new ArgsException();
            }
          }
        
          public void set(String s) {
          }
        
          public Object get() {
            return stringValue;
          }
        }
        
        private class IntegerArgumentMarshaler extends ArgumentMarshaler {
          private int intValue = 0;
        
          public void set(Iterator<String> currentArgument) throws ArgsException {
            String parameter = null;
            try {
              parameter = currentArgument.next();
              set(parameter);
            } catch (NoSuchElementException e) {
              errorCode = ErrorCode.MISSING_INTEGER;
              throw new ArgsException();
            } catch (ArgsException e) {
              errorParameter = parameter;
              errorCode = ErrorCode.INVALID_INTEGER;
              throw e;
            }
          }
        
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


    이제 인수 유형을 일일이 확인하던 코드도 제거한다.
    
    ```java
    private boolean setArgument(char argChar) throws ArgsException {
      ArgumentMarshaler m = marshalers.get(argChar);
      if (m == null)
        return false;
      try {
        m.set(currentArgument);
        return true;
      } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
      }
    }
    ```
    
    IntegerArgumentMarshaler 안을 정리하자.
    
    ```jsx
    private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    	private int intValue = 0
    	public void set(Iterator<String> currentArgument) throws ArgsException {
    		String parameter = null;
    		try {
    			parameter = currentArgument.next();
    			intValue = Integer.parseInt(parameter);
    		} catch (NoSuchElementException e) {
    			errorCode = ErrorCode.MISSING_INTEGER;
    			throw new ArgsException();
    		} catch (NumberFormatException e) {
    			errorParameter = parameter;
    			errorCode = ErrorCode.INVALID_INTEGER;
    			throw new ArgsException();
    		}
    	}
    	public Object get() {
    		return intValue;
    	}
    }
    ```
    
    ArgumentMarshaler를 인터페이스로 변경한다.
    
    ```jsx
    private interface ArgumentMarshaler {
    	void set(Iterator<String> currentArgument) throws ArgsException;
    	Object get();
    }
    ```
    
    새로운 인수 유형을 넣기 쉬운 구조로 변경되었다. 새로운 것을 넣기에 아주 약간의 변화만이 필요하며, 이 변화들은 독립적이다. double 유형으로 확인해보자.
    
    ```jsx
    public void testSimpleDoublePresent() throws Exception {
    	Args args = new Args("x##", new String[] {"-x","42.3"});
    	assertTrue(args.isValid());
    	assertEquals(1, args.cardinality());
    	assertTrue(args.has('x'));
    	assertEquals(42.3, args.getDouble('x'), .001);
    }
    ```
    
    스키마 파싱 코드를 정리하고, double 인수 타입을 위한 ## 발견 로직을 추가하자.
    
    ```jsx
    private void parseSchemaElement(String element) throws ParseException {
    	char elementId = element.charAt(0);
    	String elementTail = element.substring(1);
    	validateSchemaElementId(elementId);
    	if (elementTail.length() == 0)
    		marshalers.put(elementId, new BooleanArgumentMarshaler());
    	else if (elementTail.equals("*"))
    		marshalers.put(elementId, new StringArgumentMarshaler());
    	else if (elementTail.equals("#"))
    		marshalers.put(elementId, new IntegerArgumentMarshaler());
    	else if (elementTail.equals("##"))
    		marshalers.put(elementId, new DoubleArgumentMarshaler());
    	else
    		throw new ParseException(String.format(
    			"Argument: %c has invalid format: %s.", elementId, elementTail), 0);
    }
    ```
    
    그리고 DoubleArgumentMarshaler 클래스를 추가한다.
    
    ```java
    private class DoubleArgumentMarshaler implements ArgumentMarshaler {
    	private double doubleValue = 0;
    	public void set(Iterator<String> currentArgument) throws ArgsException {
    		String parameter = null;
    		try {
    			parameter = currentArgument.next();
    			doubleValue = Double.parseDouble(parameter);
    		} catch (NoSuchElementException e) {
    			errorCode = ErrorCode.MISSING_DOUBLE;
    			throw new ArgsException();
    		} catch (NumberFormatException e) {
    			errorParameter = parameter;
    			errorCode = ErrorCode.INVALID_DOUBLE;
    			throw new ArgsException();
    		}
    	}
    	public Object get() {
    		return doubleValue;
    	}
    }
    
    // error enum
    private enum ErrorCode {
    	OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT, 
    	MISSING_DOUBLE, INVALID_DOUBLE
    }
    
    // getDouble function
    public double getDouble(char arg) {
    	Args.ArgumentMarshaler am = marshalers.get(arg);
    	try {
    		return am == null ? 0 : (Double) am.get();
    	} catch (Exception e) {
    		return 0.0;
    	}
    }
    ```
    
    손쉽게 모든 테스트가 통과한다. 다음 테스트 케스로 에러 처리가 올바른지 확인하자. 
    
    ```java
    // 구문분석이 불가능한 string을 double 인수에 전달한다.
    public void testInvalidDouble() throws Exception {
    	Args args = new Args("x##", new String[] {"-x","Forty two"});
    	assertFalse(args.isValid());
    	assertEquals(0, args.cardinality());
    	assertFalse(args.has('x'));
    	assertEquals(0, args.getInt('x'));
    	assertEquals("Argument -x expects a double but was 'Forty two'.", 
    	args.errorMessage());
    }
    	---
    public String errorMessage() throws Exception {
    	switch (errorCode) {
    		//...
    		case INVALID_DOUBLE:
    			return String.format("Argument -%c expects a double but was '%s'.",
    				errorArgumentId, errorParameter);
    		case MISSING_DOUBLE:
    			return String.format("Could not find double parameter for -%c.", 
    				errorArgumentId);
    	}
    	return "";
    }
    
    // 다음은 double 인수를 빠뜨린 경우다.
    public void testMissingDouble() throws Exception {
    	Args args = new Args("x##", new String[]{"-x"});
    	assertFalse(args.isValid());
    	assertEquals(0, args.cardinality());
    	assertFalse(args.has('x'));
    	assertEquals(0.0, args.getDouble('x'), 0.01);
    	assertEquals("Could not find double parameter for -x.", 
    	args.errorMessage());
    }
    ```
    
    예외 코드는 보기 싫고 Args 클래스에 속하지 않는다. 게다가 ParseException을 던지지만 ParseException도 Args 클래스에 속하지 않는다. 이들을 모아 ArgsException 클래스로 이동시킨다.
    
    ```java
    public class ArgsException extends Exception {
        private char errorArgumentId = '\0';
        private String errorParameter = "TILT";
        private ErrorCode errorCode = ErrorCode.OK;
        public ArgsException() {}
        public ArgsException(String message) {super(message);}
        public enum ErrorCode {
          OK, MISSING_STRING, MISSING_INTEGER, INVALID_INTEGER, UNEXPECTED_ARGUMENT,
          MISSING_DOUBLE, INVALID_DOUBLE}
      }
    ---
      public class Args {
     ...
        private char errorArgumentId = '\0';
        private String errorParameter = "TILT";
        private ArgsException.ErrorCode errorCode = ArgsException.ErrorCode.OK;
        private List<String> argsList;
        public Args(String schema, String[] args) throws ArgsException {
          this.schema = schema;
          argsList = Arrays.asList(args);
          valid = parse();
        }
        private boolean parse() throws ArgsException {
          if (schema.length() == 0 && argsList.size() == 0)
            return true;
          parseSchema();
          try {
            parseArguments();
          } catch (ArgsException e) {
          }
          return valid;
        }
        private boolean parseSchema() throws ArgsException { ...
        }
        private void parseSchemaElement(String element) throws ArgsException { ...
     else
          throw new ArgsException(
              String.format("Argument: %c has invalid format: %s.",
                  elementId,elementTail));
        }
        private void validateSchemaElementId(char elementId) throws ArgsException {
          if (!Character.isLetter(elementId)) {
            throw new ArgsException(
                "Bad character:" + elementId + "in Args format: " + schema);
          }
        }
     ...
        private void parseElement(char argChar) throws ArgsException {
          if (setArgument(argChar))
            argsFound.add(argChar);
          else {
            unexpectedArguments.add(argChar);
            errorCode = ArgsException.ErrorCode.UNEXPECTED_ARGUMENT;
            valid = false;
          }
        }
     ...
        private class StringArgumentMarshaler implements ArgumentMarshaler {
          private String stringValue = "";
          public void set(Iterator<String> currentArgument) throws ArgsException {
            try {
              stringValue = currentArgument.next();
            } catch (NoSuchElementException e) {
              errorCode = ArgsException.ErrorCode.MISSING_STRING;
              throw new ArgsException();
            }
          }
          public Object get() {
            return stringValue;
          }
        }
        private class IntegerArgumentMarshaler implements ArgumentMarshaler {
          private int intValue = 0;
          public void set(Iterator<String> currentArgument) throws ArgsException {
            String parameter = null;
            try {
              parameter = currentArgument.next();
              intValue = Integer.parseInt(parameter);
            } catch (NoSuchElementException e) {
              errorCode = ArgsException.ErrorCode.MISSING_INTEGER;
              throw new ArgsException();
            } catch (NumberFormatException e) {
              errorParameter = parameter;
              errorCode = ArgsException.ErrorCode.INVALID_INTEGER;
              throw new ArgsException();
            }
          }
          public Object get() {
            return intValue;
          }
        }
        private class DoubleArgumentMarshaler implements ArgumentMarshaler {
          private double doubleValue = 0;
          public void set(Iterator<String> currentArgument) throws ArgsException {
            String parameter = null;
            try {
              parameter = currentArgument.next();
              doubleValue = Double.parseDouble(parameter);
            } catch (NoSuchElementException e) {
              errorCode = ArgsException.ErrorCode.MISSING_DOUBLE;
              throw new ArgsException();
            } catch (NumberFormatException e) {
              errorParameter = parameter;
              errorCode = ArgsException.ErrorCode.INVALID_DOUBLE;
              throw new ArgsException();
            }
          }
          public Object get() {
            return doubleValue;
          }
        }
      }
    ```
    
    이제 Args 클래스가 던지는 예외는 ArgsException 뿐이다. ArgException을 독자 모듈로 만들어 잡다한 오류 지원 코드를 옮겨올 수 있다. 그리고 Args의 확장도 쉬워진다.


## 최종

- ArgsTest.java

    ```java
    public class ArgsTest extends TestCase {
        public void testCreateWithNoSchemaOrArguments() throws Exception {
          Args args = new Args("", new String[0]);
          assertEquals(0, args.cardinality());
        }
        public void testWithNoSchemaButWithOneArgument() throws Exception {
          try {
            new Args("", new String[]{"-x"});
            fail();
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT,
                e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
          }
        }
        public void testWithNoSchemaButWithMultipleArguments() throws Exception {
          try {
            new Args("", new String[]{"-x", "-y"});
            fail();
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT,
                e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
          }
        }
        public void testNonLetterSchema() throws Exception {
          try {
            new Args("*", new String[]{});
            fail("Args constructor should have thrown exception");
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME,
                e.getErrorCode());
            assertEquals('*', e.getErrorArgumentId());
          }
        }
        public void testInvalidArgumentFormat() throws Exception {
          try {
            new Args("f~", new String[]{});
            fail("Args constructor should have throws exception");
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_FORMAT, e.getErrorCode());
            assertEquals('f', e.getErrorArgumentId());
          }
        }
        public void testSimpleBooleanPresent() throws Exception {
          Args args = new Args("x", new String[]{"-x"});
          assertEquals(1, args.cardinality());
          assertEquals(true, args.getBoolean('x'));
        }
        public void testSimpleStringPresent() throws Exception {
          Args args = new Args("x*", new String[]{"-x", "param"});
          assertEquals(1, args.cardinality());
          assertTrue(args.has('x'));
          assertEquals("param", args.getString('x'));
        }
        public void testMissingStringArgument() throws Exception {
          try {
            new Args("x*", new String[]{"-x"});
            fail();
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_STRING, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
          }
        }
        public void testSpacesInFormat() throws Exception {
          Args args = new Args("x, y", new String[]{"-xy"});
          assertEquals(2, args.cardinality());
          assertTrue(args.has('x'));
          assertTrue(args.has('y'));
        }
        public void testSimpleIntPresent() throws Exception {
          Args args = new Args("x#", new String[]{"-x", "42"});
          assertEquals(1, args.cardinality());
          assertTrue(args.has('x'));
          assertEquals(42, args.getInt('x'));
        }
        public void testInvalidInteger() throws Exception {
          try {
            new Args("x#", new String[]{"-x", "Forty two"});
            fail();
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_INTEGER, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
            assertEquals("Forty two", e.getErrorParameter());
          }
        }
        public void testMissingInteger() throws Exception {
          try {
            new Args("x#", new String[]{"-x"});
            fail();
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_INTEGER, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
          }
        }
        public void testSimpleDoublePresent() throws Exception {
          Args args = new Args("x##", new String[]{"-x", "42.3"});
          assertEquals(1, args.cardinality());
          assertTrue(args.has('x'));
          assertEquals(42.3, args.getDouble('x'), .001);
        }
        public void testInvalidDouble() throws Exception {
          try {
            new Args("x##", new String[]{"-x", "Forty two"});
            fail();
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_DOUBLE, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
            assertEquals("Forty two", e.getErrorParameter());
          }
        }
        public void testMissingDouble() throws Exception {
          try {
            new Args("x##", new String[]{"-x"});
            fail();
          } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.MISSING_DOUBLE, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
          }
        }
      }
    ```

- ArgsExceptionTest.java

    ```java
    public class ArgsExceptionTest extends TestCase {
        public void testUnexpectedMessage() throws Exception {
          ArgsException e =new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT,
              'x', null);
          assertEquals("Argument -x unexpected.", e.errorMessage());
        }
        public void testMissingStringMessage() throws Exception {
          ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_STRING,
              'x', null);
          assertEquals("Could not find string parameter for -x.", e.errorMessage());
        }
        public void testInvalidIntegerMessage() throws Exception {
          ArgsException e =
              new ArgsException(ArgsException.ErrorCode.INVALID_INTEGER,
                  'x', "Forty two");
          assertEquals("Argument -x expects an integer but was 'Forty two'.",
              e.errorMessage());
        }
        public void testMissingIntegerMessage() throws Exception {
          ArgsException e =
              new ArgsException(ArgsException.ErrorCode.MISSING_INTEGER, 'x', null);
          assertEquals("Could not find integer parameter for -x.", e.errorMessage());
        }
        public void testInvalidDoubleMessage() throws Exception {
          ArgsException e = new ArgsException(ArgsException.ErrorCode.INVALID_DOUBLE,
              'x', "Forty two");
          assertEquals("Argument -x expects a double but was 'Forty two'.",
              e.errorMessage());
        }
        public void testMissingDoubleMessage() throws Exception {
          ArgsException e = new ArgsException(ArgsException.ErrorCode.MISSING_DOUBLE,
              'x', null);
          assertEquals("Could not find double parameter for -x.", e.errorMessage());
        }
      }
    ```

- ArgsException.java

    ```java
    public class ArgsException extends Exception {
        private char errorArgumentId = '\0';
        private String errorParameter = "TILT";
        private ErrorCode errorCode = ErrorCode.OK;
        public ArgsException() {}
        public ArgsException(String message) {super(message);}
        public ArgsException(ErrorCode errorCode) {
          this.errorCode = errorCode;
        }
        public ArgsException(ErrorCode errorCode, String errorParameter) {
          this.errorCode = errorCode;
          this.errorParameter = errorParameter;
        }
        public ArgsException(ErrorCode errorCode, char errorArgumentId,
                             String errorParameter) {
          this.errorCode = errorCode;
          this.errorParameter = errorParameter;
          this.errorArgumentId = errorArgumentId;
        }
        public char getErrorArgumentId() {
          return errorArgumentId;
        }
        public void setErrorArgumentId(char errorArgumentId) {
          this.errorArgumentId = errorArgumentId;
        }
        public String getErrorParameter() {
          return errorParameter;
        }
        public void setErrorParameter(String errorParameter) {
          this.errorParameter = errorParameter;
        }
        public ErrorCode getErrorCode() {
          return errorCode;
        }
        public void setErrorCode(ErrorCode errorCode) {
          this.errorCode = errorCode;
        }
        public String errorMessage() throws Exception {
          switch (errorCode) {
            case OK:
              throw new Exception("TILT: Should not get here.");
            case UNEXPECTED_ARGUMENT:
              return String.format("Argument -%c unexpected.", errorArgumentId);
            case MISSING_STRING:
              return String.format("Could not find string parameter for -%c.",
                  errorArgumentId);
            case INVALID_INTEGER:
              return String.format("Argument -%c expects an integer but was '%s'.",
                  errorArgumentId, errorParameter);
            case MISSING_INTEGER:
              return String.format("Could not find integer parameter for -%c.",
                  errorArgumentId);
            case INVALID_DOUBLE:
              return String.format("Argument -%c expects a double but was '%s'.",
                  errorArgumentId, errorParameter);
            case MISSING_DOUBLE:
              return String.format("Could not find double parameter for -%c.",
                  errorArgumentId);
          }
          return "";
        }
        public enum ErrorCode {
          OK, INVALID_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME,
          MISSING_STRING,
          MISSING_INTEGER, INVALID_INTEGER,
          MISSING_DOUBLE, INVALID_DOUBLE}
      }
    ```

- Args.java

    ```java
    public class Args {
        private String schema;
        private Map<Character, ArgumentMarshaler> marshalers =
            new HashMap<Character, ArgumentMarshaler>();
        private Set<Character> argsFound = new HashSet<Character>();
        private Iterator<String> currentArgument;
        private List<String> argsList;
        public Args(String schema, String[] args) throws ArgsException {
          this.schema = schema;
          argsList = Arrays.asList(args);
          parse();
        }
        private void parse() throws ArgsException {
          parseSchema();
          parseArguments();
        }
        private boolean parseSchema() throws ArgsException {
          for (String element : schema.split(",")) {
            if (element.length() > 0) {
              parseSchemaElement(element.trim());
            }
          }
          return true;
        }
        private void parseSchemaElement(String element) throws ArgsException {
          char elementId = element.charAt(0);
          String elementTail = element.substring(1);
          validateSchemaElementId(elementId);
          if (elementTail.length() == 0)
            marshalers.put(elementId, new BooleanArgumentMarshaler());
          else if (elementTail.equals("*"))
            marshalers.put(elementId, new StringArgumentMarshaler());
          else if (elementTail.equals("#"))
            marshalers.put(elementId, new IntegerArgumentMarshaler());
          else if (elementTail.equals("##"))
            marshalers.put(elementId, new DoubleArgumentMarshaler());
          else
            throw new ArgsException(ArgsException.ErrorCode.INVALID_FORMAT,
                elementId, elementTail);
        }
        private void validateSchemaElementId(char elementId) throws ArgsException {
          if (!Character.isLetter(elementId)) {
            throw new ArgsException(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME,
                elementId, null);
          }
        }
        private void parseArguments() throws ArgsException {
          for (currentArgument = argsList.iterator(); currentArgument.hasNext();) {
            String arg = currentArgument.next();
            parseArgument(arg);
          }
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
            throw new ArgsException(ArgsException.ErrorCode.UNEXPECTED_ARGUMENT,
                argChar, null);
          }
        }
        private boolean setArgument(char argChar) throws ArgsException {
          ArgumentMarshaler m = marshalers.get(argChar);
          if (m == null)
            return false;
          try {
            m.set(currentArgument);
            return true;
          } catch (ArgsException e) {
            e.setErrorArgumentId(argChar);
            throw e;
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
        public boolean getBoolean(char arg) {
          ArgumentMarshaler am = marshalers.get(arg);
          boolean b = false;
          try {
            b = am != null && (Boolean) am.get();
          } catch (ClassCastException e) {
            b = false;
          }
          return b;
        }
        public String getString(char arg) {
          ArgumentMarshaler am = marshalers.get(arg);
          try {
            return am == null ? "" : (String) am.get();
          } catch (ClassCastException e) {
            return "";
          }
        }
        public int getInt(char arg) {
          ArgumentMarshaler am = marshalers.get(arg);
          try {
            return am == null ? 0 : (Integer) am.get();
          } catch (Exception e) {
            return 0;
          }
        }
        public double getDouble(char arg) {
          ArgumentMarshaler am = marshalers.get(arg);
          try {
            return am == null ? 0 : (Double) am.get();
          } catch (Exception e) {
            return 0.0;
          }
        }
        public boolean has(char arg) {
          return argsFound.contains(arg);
        }
      }
    ```

- Args 클래스에서는 주로 코드 제거. ArgsException으로 상당량 이동.
- ArgumentMarshaler 클래스도 각자 파일로 이동.

소프트웨어 설계 품질이 높이기

- 적절하게 코드 분리하기
- 관심사 분리하기

눈여겨 볼 코드는 ArgsException의 errorMessage 메서드이다. Args에 있던 이 메서드는  SRP 위반. Args가 오류 메시지 형식까지 책임졌기 때문이다. 그치만 ArgsException이 오류 메시지 형식을 처리해야하나? → 이것은 절충안으로, 깔끔하게 만들어진 오류 메시지로 얻는 장점은 무시하기 어렵다. 엄밀하게는 새로운 클래스가 필요하다.

# 결론

그저 돌아가는 코드만으로는 부족하다. 나쁜 코드는 썩어 문드러진다. 점점 속도가 느려진다.

깨끗한 코드로 개선할 수 있지만, 비용이 많이든다.

처음부터 깨끗하게 유지하기가 상대적으로 쉽다.
