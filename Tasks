import Foundation

// Define a class to represent an asynchronous task
class Task {
    let taskID: String
    let executionTime: TimeInterval
    var completion: (Result<Void, Error>) -> Void
    var retryCount: Int
    
    init(taskID: String, executionTime: TimeInterval, completion: @escaping (Result<Void, Error>) -> Void) {
        self.taskID = taskID
        self.executionTime = executionTime
        self.completion = completion
        self.retryCount = 0
    }
    
    // Execute the task asynchronously
    func execute() {
        print("Executing task \(taskID)")
        
        DispatchQueue.global().asyncAfter(deadline: .now() + executionTime) {
            // Simulate success or failure randomly
            let success = Bool.random()
            
            if success {
                self.completion(.success(()))
            } else {
                self.completion(.failure(NSError(domain: "TaskError", code: 1, userInfo: [NSLocalizedDescriptionKey: "Task failed"])))
            }
        }
    }
}

// Define a class to represent a meta-task
class MetaTask {
    var tasks: [Task]
    let retryLimit: Int
    let timeout: TimeInterval
    var results: [Result<Void, Error>] = []
    var timer: Timer?
    
    init(tasks: [Task], retryLimit: Int, timeout: TimeInterval) {
        self.tasks = tasks
        self.retryLimit = retryLimit
        self.timeout = timeout
    }
    
    // Execute all tasks in the meta-task
    func execute(completion: @escaping () -> Void) {
        let dispatchGroup = DispatchGroup()
        
        // Start timer
        timer = Timer.scheduledTimer(withTimeInterval: timeout, repeats: false) { [weak self] _ in
            self?.handleTimeout()
        }
        
        // Execute tasks
        for task in tasks {
            dispatchGroup.enter()
            task.execute()
            task.completion = { result in
                self.results.append(result)
                dispatchGroup.leave() // Leave the DispatchGroup after task completion
            }
        }
        
        // Notify when all tasks are completed
        dispatchGroup.notify(queue: .main) { [weak self] in
            self?.timer?.invalidate()
            print("All tasks completed successfully")
            completion()
        }
    }
    
    // Handle timeout
    private func handleTimeout() {
        print("MetaTask timed out")
        
        for task in tasks {
            if task.retryCount < retryLimit {
                print("Retrying task \(task.taskID)")
                task.execute()
                task.retryCount += 1
            } else {
                print("Retry limit reached for task \(task.taskID)")
                task.completion(.failure(NSError(domain: "RetryLimitError", code: 2, userInfo: [NSLocalizedDescriptionKey: "Retry limit reached"])))
            }
        }
    }
}

// Define a class to manage tasks and meta-tasks
class TaskManager {
    // Execute a meta-task
    func execute(metaTask: MetaTask, completion: @escaping () -> Void) {
        metaTask.execute {
            completion()
        }
    }
}

// Example usage

// Create a meta-task
let metaTask = MetaTask(tasks: [], retryLimit: 3, timeout: 5.0)

// Create tasks
let task1 = Task(taskID: "Task 1", executionTime: 2.0) { result in
    switch result {
    case .success:
        print("Task 1 succeeded")
        // Perform actions for success
    case .failure(let error):
        print("Task 1 failed: \(error.localizedDescription)")
        // Perform actions for failure
    }
}

let task2 = Task(taskID: "Task 2", executionTime: 3.0) { result in
    switch result {
    case .success:
        print("Task 2 succeeded")
        // Perform actions for success
    case .failure(let error):
        print("Task 2 failed: \(error.localizedDescription)")
        // Perform actions for failure
    }
}

let failingTask = Task(taskID: "Failing Task", executionTime: 3.0) { result in
    switch result {
    case .success:
        print("Failing Task succeeded")
        // Perform actions for success
    case .failure(let error):
        print("Failing Task failed: \(error.localizedDescription)")
        // Perform actions for failure
    }
}


// Assign tasks to the meta-task
metaTask.tasks = [task1, task2, failingTask]

// Execute the meta-task
let taskManager = TaskManager()
taskManager.execute(metaTask: metaTask) {
    // Print the results after all tasks are completed
    print("Results:")
    for (index, result) in metaTask.results.enumerated() {
        switch result {
        case .success:
            print("Task \(index + 1): Success")
        case .failure(let error):
            print("Task \(index + 1): Failure - \(error.localizedDescription)")
        }
    }
}

