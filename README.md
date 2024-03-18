# Asynchronous Task and MetaTask Manager

## Purpose
The Asynchronous Task and MetaTask Manager framework simplifies the execution of multiple asynchronous tasks concurrently. It provides functionality for managing tasks, handling retries, managing timeouts, and tracking task results within a meta-task.

## Components
### Task
Represents an asynchronous task with the following attributes:
- **taskID**: Unique identifier for the task.
- **executionTime**: Time interval specifying how long the task takes to execute.
- **completion**: Closure to handle the task completion, returning a `Result<Void, Error>`.

### MetaTask
Represents a collection of tasks to be executed together, offering the following features:
- **tasks**: An array of tasks to be executed concurrently.
- **retryLimit**: Maximum number of retries allowed for each task.
- **timeout**: Time interval specifying the duration after which the meta-task should timeout.
- **results**: An array to store the results of each task execution.
- **timer**: Timer to handle timeouts.

### TaskManager
Manages the execution of individual tasks and meta-tasks, providing the following functionality:
- **execute(metaTask:completion:)**: Executes a given meta-task, invoking the completion handler when all tasks are completed.

## Usage
1. **Task Creation**: Create instances of the `Task` class, specifying task details such as ID, execution time, and completion handler.
2. **MetaTask Creation**: Create a `MetaTask` instance, passing an array of tasks, retry count, and timeout duration.
3. **Execution**: Use the `TaskManager` to execute individual tasks or meta-tasks.

## Problem Solved
- **Asynchronous Task Management**: Simplifies the execution of multiple asynchronous tasks concurrently.
- **Retry Handling**: Provides automatic retry functionality for tasks that fail.
- **Timeout Management**: Ensures tasks complete within a specified time frame, handling timeouts gracefully.
- **Result Tracking**: Tracks the results of each task within a meta-task, enabling further analysis or action based on task outcomes.

## Example Usage
```swift
// Create tasks
let task1 = Task(taskID: "Task 1", executionTime: 2.0) { result in /* Completion handler */ }
let task2 = Task(taskID: "Task 2", executionTime: 3.0) { result in /* Completion handler */ }
let failingTask = Task(taskID: "Failing Task", executionTime: 3.0) { result in /* Completion handler */ }

// Create a meta-task
let metaTask = MetaTask(tasks: [task1, task2, failingTask], retryLimit: 3, timeout: 5.0)

// Execute the meta-task
let taskManager = TaskManager()
taskManager.execute(metaTask: metaTask) {
    // Print the results after all tasks are completed
    for (index, result) in metaTask.results.enumerated() {
        switch result {
        case .success:
            print("Task \(index + 1): Success")
        case .failure(let error):
            print("Task \(index + 1): Failure - \(error.localizedDescription)")
        }
    }
}
