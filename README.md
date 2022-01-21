### Networking Step 1, URLSession

```swift
import UIKit
import Combine

func getPosts() -> AnyPublisher<Data, URLError> {
    guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else {
        fatalError("Invalid URL")
    }
    return URLSession.shared.dataTaskPublisher(for: url)
        .map { $0.data }
        .eraseToAnyPublisher()
}

let cancellable = getPosts().sink { _ in } receiveValue: { print($0)}
```

### Networking Step 2, Codable

```swift
import UIKit
import Combine

struct Post: Codable {
    let title: String
    let body: String
}

func getPosts() -> AnyPublisher<[Post], Error> {
    guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else {
        fatalError("Invalid URL")
    }
    return URLSession.shared.dataTaskPublisher(for: url)
        .map { $0.data }
        .decode(type: [Post].self, decoder: JSONDecoder())
        .eraseToAnyPublisher()
}

let cancellable = getPosts().sink { _ in } receiveValue: { print($0)}
```

### Networking Step 3, With UI

```swift
import UIKit
import Combine

struct Post: Codable {
    let title: String
    let body: String
}

class Webservice {
    func getPosts() -> AnyPublisher<[Post], Error> {
        guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else {
            fatalError("Invalid URL")
        }
        return URLSession.shared.dataTaskPublisher(for: url)
            .map { $0.data }
            .decode(type: [Post].self, decoder: JSONDecoder())
            .receive(on: RunLoop.main)
            .eraseToAnyPublisher()
    }

}

class PostlistTableViewController: UITableViewController {

    private var webservice = Webservice()
    private var cancellable: AnyCancellable?
    private var posts = [Post]() {
        didSet {
            self.tableView.reloadData()
        }
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        self.cancellable = self.webservice.getPosts()
            .catch { _ in Just([Post]())} // Empty Post
            .assign(to: \.posts, on: self)
    }

    override func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.posts.count
    }

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "PostTableViewCell", for: indexPath)
        let post = self.posts[indexPath.row]
        cell.textLabel?.text = post.title
        return cell
    }

}
```
