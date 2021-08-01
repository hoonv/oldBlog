---
layout: post
title:  "Welcome to Jekyll!"
date:   2021-08-05 22:45:14 +0900
categories: asdfasdf
---
안녕하세요 채훈기 입니다.
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:
``` javascript
var hello = function(){
  var radio = document.getElementsByName('gender');
  for(i=0; i < radio.length; i++){
    if(radio[i].checked){
      var result = radio[i].value;
    }
  }
  return result;
} 
```

``` swift

import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        fetchData()
    }
    
    @IBOutlet weak var collectionView: UICollectionView!

    let imageLoader = ImageLoader()
    var datas: [Passenger] = []
    var page = 0

    func fetchData() {
        let urlString = "https://api.instantwebtools.net/v1/passenger?page=\(page)&size=20"
        let url = URL(string: urlString)!
        URLSession.shared.dataTask(with: url) { data, _, _ in
            guard let data = data,
                  let json = try? JSONDecoder().decode(PassengerData.self, from: data)
            else {
                print("fetch fail")
                return
            }
            self.datas.append(contentsOf: json.data)
            self.insertIndexPathes(self.datas.count-20..<self.datas.count)
        }.resume()
        page += 1
    }
    
    //reloadData하지 않고 insertItem으로 새로 추가된 data를 추가함.
    func insertIndexPathes(_ range: Range<Int>) {
        let indexes = range.map { IndexPath(row: $0, section: 0) }
        DispatchQueue.main.async {
            self.collectionView.performBatchUpdates({
                self.collectionView.insertItems(at: indexes)
            }, completion: nil)
        }
    }
    
    let layout1: UICollectionViewFlowLayout = {
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 380, height: 150.0)
        return layout
    }()
    
    let layout2: UICollectionViewFlowLayout = {
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 150, height: 150)
        return layout
    }()
    
    var flag = false
}

extension ViewController {

    @objc func refreshHandler() {

        DispatchQueue.main.async {
            let layout = self.flag ? self.layout1 : self.layout2
            self.flag.toggle()
            self.collectionView.setCollectionViewLayout(layout, animated: true)
            self.collectionView.refreshControl?.endRefreshing()
        }
    }
    func setupUI() {
        collectionView.setCollectionViewLayout(layout1, animated: true)
        collectionView.refreshControl = UIRefreshControl()
        collectionView.refreshControl?.tintColor = .red
        collectionView.refreshControl?.addTarget(self, action: #selector(refreshHandler), for: .valueChanged)
        collectionView.register(Cell.self, forCellWithReuseIdentifier: "Cell")
    }
}

extension ViewController: UICollectionViewDelegate, UICollectionViewDataSource, UICollectionViewDelegateFlowLayout {

    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return datas.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath) as? Cell else { return Cell() }
        let data = datas[indexPath.row]
        cell.title.text = data.name
        cell.id.text = data.id
        let uuid = imageLoader.loadImage(indexPath) {
            let image = try? $0.get()
            DispatchQueue.main.async {
                cell.imageView.image = image
            }
        }
        // uuid를 받아 클로져를 등록해둠, 후에 dequeue되면 prepare에서 이 uuid를 사용해 취소할것
        cell.onReuse = {
          if let token = uuid {
            self.imageLoader.cancelLoad(token)
          }
        }
        return cell
    }
    
    // 마지막 Cell이 Will Display될대 fetch함
    func collectionView(_ collectionView: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt indexPath: IndexPath) {
         if (indexPath.row == datas.count - 1 ) {
            fetchData()
         }
    }
}

```
{% highlight swift %}
    @objc func refreshHandler() {

        DispatchQueue.main.async {
            let layout = self.flag ? self.layout1 : self.layout2
            self.flag.toggle()
            self.collectionView.setCollectionViewLayout(layout, animated: true)
            self.collectionView.refreshControl?.endRefreshing()
        }
    }
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
