---
title: "Spec Structure 101: Your Guide to Readable, Maintainable Tests"
date: 2024-05-22 13:18:00 +0000
categories: [Ruby on Rails, Layered Design for Ruby on Rails Applications]
tags: [rails, ui, view components, rendering, parameters, development, web development]
---

**Bad specs are a ticking time bomb. They are hurting your code’s maintainability.** As your application grows, tests get more disorganized. They become harder to navigate, understand, and change. Before long, you find yourself lost in a labyrinth. It is full of duplicated code, hidden intent, and twisted logic. To avoid this nightmare, it’s crucial to structure your code examples with care from the outset.

## Structuring Code Examples in Your Specs
When you write tests for your application, you must keep a well-organized codebase. Poorly structured specs can become a nightmare. They are hard to read, understand, and maintain. They cause confusion and wasted time.

### Get the Words Right
One key idea is to pick clear and meaningful names for your `describe` and `it` blocks. RSpec provides many aliases. They help you show the intent of your examples more clearly.

```ruby
RSpec.describe LMS::Course do
  context "when the course has no assignments" do
    example "shows a placeholder message" do
      course = LMS::Course.create(name: "Introduction to Ruby")
      expect(course.assignments).to be_empty
      expect(course.assignment_list).to eq("No assignments have been added yet.")
    end
  end
end
```

In the example above, we use `context` to describe the scenario under test. We use `example` to explain the expected behavior. This approach makes the spec easier to read. It also shows the intent behind the test case well.

### Sharing Common Logic
As your test suite grows, you’ll find times when you need to share setup logic. RSpec provides many tools to help you with this. They include let definitions, hooks, and helper methods.

### Let Definitions
`let` definitions allow you to define memoized helper methods that are lazily evaluated. This means that the code within the `let` block won't be executed until it's referenced within an example or another `let` definition.

```ruby
RSpec.describe LMS::GradeCalculator do
  let(:course) { LMS::Course.create(name: "Intro to Ruby") }
  let(:student) { LMS::Student.create(name: "John Doe") }
  let(:assignments) do
    [
      LMS::Assignment.create(name: "Assignment 1", points: 100, course: course),
      LMS::Assignment.create(name: "Assignment 2", points: 80, course: course)
    ]
  end

  # ...
end
```

In this example, we define `let` blocks for a course, a student, and a list of assignments. You can use these helper methods in the whole example group. They cut duplication and improve readability.

### Hooks
Hooks allow you to define code that should run before or after each example or example group. They are particularly useful when you need to set up or tear down shared state that cannot be easily encapsulated within `let` definitions.

```ruby
RSpec.describe LMS::GradeCalculator do
  let(:course) { LMS::Course.create(name: "Intro to Ruby") }
  let(:student) { LMS::Student.create(name: "John Doe") }

  before do
    course.enroll(student)
  end

  # ...
end
```

In this example, we use a `before` hook to ensure that the student is enrolled in the course before each example is run. This setup code will be executed before every example within the example group, ensuring a consistent starting state.

### Helper Methods
Sometimes, your setup logic can become too complex to fit neatly into `let` definitions or hooks. In such cases, you can define helper methods within your example group to encapsulate the shared logic.

```ruby
RSpec.describe LMS::GradeCalculator do
  def create_course_with_assignments(name, assignments)
    course = LMS::Course.create(name: name)
    assignments.each do |assignment|
      course.assignments << LMS::Assignment.create(assignment)
    end
    course
  end

  # ...
end
```

Here, we define a `create_course_with_assignments` helper method. It creates a course and links it with a list of assignments. This helper can be used throughout the example group, promoting code reuse and improving readability.

### Sharing Example Groups
As your test suite continues to grow, you may find yourself needing to reuse entire example groups across multiple spec files. RSpec provides shared example groups to help you do this. They let you define reusable examples, `let` constructs, and add hooks.

### Shared Examples
Shared examples are great for testing common behaviors. These behaviors span multiple parts of your application. They allow you to define a set of examples once and then include them in multiple example groups.

```ruby
RSpec.shared_examples "a gradable assignment" do
  it "calculates the grade based on points earned" do
    assignment = described_class.new(name: "Assignment 1", total_points: 100)
    assignment.submit_for_grading(points_earned: 80)
    expect(assignment.grade).to eq(80)
  end

  # ...
end

RSpec.describe LMS::Assignment do
  it_behaves_like "a gradable assignment"
  # ...
end

RSpec.describe LMS::Quiz do
  it_behaves_like "a gradable assignment"
  # ...
end
```

In this example, we define a shared example group called “a gradable assignment”. It contains examples for testing the grading of assignments and quizzes. We then include these shared examples in the `LMS::Assignment` and `LMS::Quiz` spec files using `it_behaves_like`.

### Customizing Shared Groups
Sometimes, you may need to customize the behavior of a shared example group based on the context in which it’s being used. RSpec allows you to pass arguments or blocks to `it_behaves_like` to achieve this.

```ruby
RSpec.shared_examples "an authenticated user" do |user_type|
  let(:user) { LMS::User.create(email: "user@example.com", password: "password", type: user_type) }

  before do
    login_as(user)
  end

  # ...
end

RSpec.describe LMS::CoursesController do
  it_behaves_like "an authenticated user", :student

  # ...
end

RSpec.describe LMS::InstructorsController do
  it_behaves_like "an authenticated user", :instructor

  # ...
end
```

In this example, we define a shared example group. It is called “an authenticated user.” It sets up an authenticated user based on the provided `user_type`. We then include this shared group in different controller specs. We pass in the right `user_type` argument (`student` or `instructor`). This approach allows us to reuse the shared examples while customizing the behavior based on the context.

By using these code structuring techniques, you can keep a well-organized and readable test suite. This will make it easier to understand, change, and expand as your app grows.

## TL;DR
When structuring code examples in your specs, remember these key points: Use descriptive names with `context`, `example`, and `specify` for clarity. Share setup logic across examples via let definitions, hooks, and helper methods to reduce duplication. Use shared example groups with `shared_examples` and `it_behaves_like`. They let you reuse examples and setup across many specs. Customize shared groups with arguments or blocks for context-specific behavior. Keep your specs organized, readable, and maintainable for long-term success.

> Learn more about this topic from book: [Effective Testing with RSpec 3](https://pragprog.com/titles/rspec3/effective-testing-with-rspec-3/)
{: .prompt-info }


