---
title: 'How to optimize if…else'
date: 2024-02-17T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*qGg1oeee8PeRrw6HszxltA.png"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Preemptive return

Suppose we have the following code:

```
if (condition) {
  doSomething;
} else {
  return;
}
```

For such code, we usually use a method called preemptive return to help us get rid of unnecessary else.

```
if (!condition) {
  return;
}

doSomething;
```

This method is generally suitable for if...else statements with a simple branch structure, where we can return early to eliminate some unnecessary if...else statements.

## Enumeration

Enumeration can also be used to get rid of if...else statements. For example:

```
String orderStatusDes;
if ("1".equals(orderStatus)) {
    orderStatusDes = "The order has not been paid";
} else if ("2".equals(orderStatus)) {
    orderStatusDes = "The order has been paid";
} else if ("3".equals(orderStatus)) {
    orderStatusDes = "The order has been shipped";
} else if ("4".equals(orderStatus)) {
    orderStatusDes = "The order has been received";
} else if ("5".equals(orderStatus)) {
    orderStatusDes = "The order has been reviewed";
}
```

You might ask who would write such code? But having been in the business for so long, I still see people with 5 or 6 years of work experience writing this kind of code. This type of code is perfect for enumeration.

First, define an enumeration class:

```
@Getter
@AllArgsConstructor
public enum OrderStatusEnum {
    UN_PAID("1","The order has not been paid"),
    PAIDED("2","The order has been paid"),
    SENDED("3","The order has been shipped"),
    SINGED("4","The order has been received"),
    EVALUATED("5","The order has been reviewed");

    private String status;

    private String statusDes;

    static OrderStatusEnum of(String status) {
        for (OrderStatusEnum statusEnum : OrderStatusEnum.values()) {
            if (statusEnum.getStatus().equals(status)) {
                return statusEnum;
            }
        }
        return null;
    }
}
```

With this enumeration class, the above code can be optimized to one line of code:

```
String orderStatusDes = OrderStatusEnum.of(orderStatus).getStatusDes();
```

## Optional, used for nonnull judgement
In every project, there must be some nonnull judgements. If it is null, you can throw an exception or return.

```
Order order = getOrderById(id);
if (order == null) {
    return "-1";
} else {
    return order.getOrderStatus();
}
```

For such code, we can use Optional to solve this elegantly.

```
return Optional.ofNullable(order).map(o -> o.getOrderStatus()).orElse("-1");
```

Isn’t this way very elegant and has style? Lastly, I’d like to add:

Preventing NullPointerExceptions (NPE) is a basic discipline for programmers.

## Table-driven method

The table-driven method is a way to find information in a table without the need to use too many if…else statements to get them out. For example:

```
if ("code1".equals(action)) {
    doAction1();
} else if ("code2".equals(action)) {
    doAction2();
} else if ("code3".equals(action)) {
    doAction3();
} else if ("code4".equals(action)) {
    doAction4();
} else if ("code5".equals(action)) {
    doAction5();
}
```

The optimization method is as follows:

```
Map<String, Function<?> action> actionMap = new HashMap<>();
action.put("code1",() -> {doAction1()});
action.put("code2",() -> {doAction2()});
action.put("code3",() -> {doAction3()});
action.put("code4",() -> {doAction4()});
action.put("code5",() -> {doAction5()});

// How to use
actionMap.get(action).apply();
```

## Strategy Pattern + Factory Method

The combination of Strategy Pattern + Factory Method is commonly used solution to replace if…else, it somewhat resembles the table-driven method above.

- **Extract condition blocks into a common interface, or the strategy interface.**

```
public interface ActionService {
    void doAction();
}
```

- **Define your own specific strategy implementation classes based on each logic, as follows**:

```
public class ActionService1 implements ActionService{
    public void doAction() {
        //do something
    }
}

public class ActionService2 implements ActionService{
    public void doAction() {
        //do something
    }
}
// Other strategies omitted
```

- **Factory class for unified dispatch to manage these strategies, as follows**:

```
public class ActionServiceFactory {
    private ActionServiceFactory(){

    }

    private static class SingletonHolder{
        private static ActionServiceFactory instance=new ActionServiceFactory();
    }

    public static ActionServiceFactory getInstance(){
        return SingletonHolder.instance;
    }

    private static final Map<String,ActionService> ACTION_SERVICE_MAP = new HashMap<String, ActionService>();

    static {
        ACTION_SERVICE_MAP.put("action1",new ActionService1());
        ACTION_SERVICE_MAP.put("action2",new ActionService2());
        ACTION_SERVICE_MAP.put("action3",new ActionService3());
        ACTION_SERVICE_MAP.put("action4",new ActionService4());
        ACTION_SERVICE_MAP.put("action5",new ActionService5());
    }

    public static ActionService getActionService(String actionCode) {
        ActionService actionService = ACTION_SERVICE_MAP.get(actionCode);
        if (actionService == null) {
            throw new RuntimeException("非法 actionCode");
        }
        return actionService;
    }

    public void doAction(String actionCode) {
        getActionService(actionCode).doAction();
    }
}
```

- **Use the Singleton method to implement factory class**.

```
ActionServiceFactory.getInstance().doAction("action1");
```

This way is also very elegant and is especially suitable for code blocks with many branches and complex logic. It decouples branch logic and business code, which is a very good solution.

## Chain of Responsibility Pattern

It’s unexpected that the chain of responsibility pattern can also optimize if…else, right? We can consider the chain of responsibility as a singly linked data structure, each object filters conditions in turn. If the condition meets, it executes and ends. If the condition doesn’t meet, it is passed on to the next node. If none of the objects can handle it, there is generally a final node to handle it uniformly.

Let’s still take the above example.

- **Define node to handle chain of responsibility requests**:

```
public abstract class ActionHandler {

    // successor node
    protected ActionHandler successor;

    /**
     * Handle request
     * @param actionCode
     */
    public void handler(String actionCode) {
        doHandler(actionCode);
    }

    // Set successor node
    protected ActionHandler setSuccessor(ActionHandler successor) {
        this.successor = successor;
        return this;
    }

    // Handle request
    public abstract void doHandler(String actionCode);
}
```

- **Define head and tail nodes for handling exceptions**:

```
// Head node, judge whether actionCode is empty
public class HeadHandler extends ActionHandler{

    @Override
    public void doHandler(String actionCode) {
        if (StringUtils.isBlank(actionCode)) {
            throw new RuntimeException("actionCode cannot be empty");
        }

        successor.doHandler(actionCode);
    }
}

// Tail node, directly throws exceptions. Because when it comes to the tail node, it means that the current code has no handler to handle it.
public class TailHandler extends ActionHandler{

    @Override
    public void doHandler(String actionCode) {
        throw new RuntimeException("The current code[" + actionCode + "] has no specific Handler to handle");
    }
}
```


- **Define specific implementation nodes for each node**:

```
public class ActionHandler1 extends ActionHandler{

    @Override
    public void doHandler(String actionCode) {
        if ("action1".equals(actionCode)) {
            doAction1();
        } else {
            // Pass to the next node
            successor.doHandler(actionCode);
        }
    }
}

public class ActionHandler2 extends ActionHandler{

    @Override
    public void doHandler(String actionCode) {
        if ("action2".equals(actionCode)) {
            doAction2();
        } else {
            // Pass to the next node
            successor.doHandler(actionCode);
        }
    }
}

// Other nodes are omitted
```

- **Define factory to build a complete chain of responsibility and be responsible for the scheduling**:

```
public class ActionHandlerFactory {
    
    private ActionHandler headHandler;
    
    private ActionHandlerFactory(){
        headHandler = new HeadHandler();
        ActionHandler actionHandler1 = new ActionHandler1();
        ActionHandler actionHandler2 = new ActionHandler2();
        ActionHandler actionHandler3 = new ActionHandler3();
        ActionHandler actionHandler4 = new ActionHandler4();
        ActionHandler actionHandler5 = new ActionHandler5();

        ActionHandler tailHandler = new TailHandler();
        
        // Build a complete chain of responsibility
        headHandler.setSuccessor(actionHandler1).setSuccessor(actionHandler2).setSuccessor(actionHandler3).
                setSuccessor(actionHandler4).setSuccessor(actionHandler5).setSuccessor(tailHandler);
    }

    private static class SingletonHolder{
        private static ActionHandlerFactory instance=new ActionHandlerFactory();
    }

    public static ActionHandlerFactory getInstance(){
        return SingletonHolder.instance;
    }
        
    public void doAction(String actionCode) {
        headHandler.doHandler(actionCode);
    }
}
```

**Usage**:

```
ActionHandlerFactory.getInstance().doAction("action1");
```

## Function

Function is a functional interface in Java 8. With good use of it, we can greatly simplify our code. For example, using it, we can easily get rid of our if…else. Take the following code as an example:

```
// Throw exception
if (...) {
  throw new RuntimeException("Oops, there's an exception...")
}

// if...else branch
if(...) {
  doSomething1();
} else {
  doSomething2();
}
```

Now we use Function to handle the above two pieces of code.

Handle throwing exceptions;

Define the functional interface of the form of throwing exceptions:

```
@FunctionalInterface
public interface ThrowExceptionFunction {

    /**
     * Throw exception
     * @param message
     */
    void throwMessage(String message);
}
```

This just requires one such functional interface. Moreover, the method has no return value, which is a consumer interface.

Add judgment tool class:

```
public class ValidateUtils {

    /**
     * Throw exception
     * @param flag
     * @return
     */
    public static ThrowExceptionFunction isTrue(Boolean flag) {
        return (errorMessage) -> {
            if (flag) {
                throw new RuntimeException(errorMessage);
            }
        };
    }
}

// Usage
ValidateUtils.isTrue(flag).throwMessage("Oops, there's an exception...");
```

Here are 7 ways to solve the problem of if…else. I believe there always are one or two solutions that you are quite satisfied with. Different solutions have their own advantages and disadvantages, and their own usage scenarios. We need to constantly comprehend in practice, constantly evolve in refactoring, and summarize the best refactoring solutions suitable for ourselves.
