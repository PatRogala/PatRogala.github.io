---
title: "Understanding Test Doubles in RSpec 3"
date: 2024-06-16 08:57:00 +0000
categories: [Ruby on Rails, Effective Testing with RSpec 3]
tags: [test doubles, stubs, mocks, spies, null objects, pure doubles, partial doubles, verifying doubles, stubbed constants, testing techniques, ruby, rspec, software testing, test driven development, tdd, unit testing, behavior driven development, bdd, test automation]
---

## Understanding Test Doubles in Software Development

When writing tests for software, we often need to create "test doubles" or "mocks" to simulate the behavior of real objects or dependencies. There are different types of test doubles, each serving a specific purpose.

### Types of Test Doubles
Test doubles can be classified based on their usage mode and origin.

#### Usage Modes
**Stub:** Stubs are used to simulate query methods that return a value without performing any side effects. They return predefined, canned responses.

```rb
# Stub for a LMS Course model
stub_course = double('Course', name: 'Introduction to Ruby', description: 'Learn Ruby basics')
```

**Mock:** Mocks are useful when testing command methods that perform side effects, rather than returning a value. They ensure that specific messages are received and can raise an error if the expected messages are not received.

```rb
# Mock for a LMS Enrollment service
mock_enrollment_service = double('EnrollmentService')
expect(mock_enrollment_service).to receive(:enroll).with(user, course)
```

**Null Object:** Null objects are benign test doubles that can stand in for any object, returning themselves in response to any message. They are useful when testing objects with multiple collaborators.

```rb
# Null object for a LMS User
null_user = double('User').as_null_object
null_user.enroll_in_course(course)
```

**Spy:** Spies record the messages they receive, allowing you to assert that specific messages were called with the correct arguments.

```rb
# Spy for a LMS Notification service
spy_notification_service = spy('NotificationService')
spy_notification_service.send_notification(user, 'Welcome!')
expect(spy_notification_service).to have_received(:send_notification).with(user, 'Welcome!')
```

#### Origins

In addition to understanding the usage modes of test doubles, it's essential to know their origins and types. Test doubles can be pure, partial, or verifying, each serving a different purpose.

**Pure Double:** Pure doubles are purpose-built by testing frameworks, like RSpec, and consist entirely of behavior you add to them. They are flexible and easy to use for testing code where you can pass in dependencies.

```rb
# Pure double for a LMS Course
pure_course = double('Course')
allow(pure_course).to receive(:name).and_return('Introduction to Ruby')
```

**Partial Double:** Sometimes, the code you're testing doesn't allow for easy dependency injection. In such cases, you can use partial doubles, which add mocking and stubbing behavior to existing Ruby objects.

```rb
# Partial double for the built-in Ruby Time class
allow(Time).to receive(:now).and_return(Time.new(2023, 6, 1))
```

**Verifying Double:** Verifying doubles help catch issues when the test double and the real dependency drift out of sync. They constrain the double's interface based on the actual class or object, ensuring that method changes are detected.

```rb
# Verifying double for a LMS User class
user = instance_double('User')
allow(user).to receive(:enroll_in_course)
```

**Stubbed Constant:** Test doubles also allow you to control the environment by replacing or removing Ruby constants for the duration of a test.

```rb
# Stubbing a constant for the default enrollment period
stub_const('LMS::DEFAULT_ENROLLMENT_PERIOD', 7)
```

Test doubles can combine both usage modes and origins. For example, you may have a pure double acting as a stub or a verifying double acting as a spy.

