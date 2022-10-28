# rxNote
# viewModel
```swift
class HolidaysViewModel {
    private let disposeBag = DisposeBag()
    // MARK: - Actions
    let isLoading = BehaviorSubject<Bool>(value: false)
    let selectedCountry = PublishSubject<String>()
    let selectedHoliday = PublishSubject<HolidayViewModel>()
    let chooseCountry = PublishSubject<Void>()
    
    // MARK: - Table View Model and Data Source
    var holidays = BehaviorSubject<[HolidayViewModel]>(value: [])
    
    // MARK: - API Call
    func fetchHolidays(onError: @escaping (String) -> ()) {
        
        self.selectedCountry
            .subscribe(onNext: { [weak self] (country) in
                guard let `self` = self else { return }
                
                self.isLoading.onNext(true)
                HolidaysService.shared.getHolidays(country: country, success: { (code, holidays) in
                    self.isLoading.onNext(false)
                    
                    let holidayItems = holidays.holidays!.compactMap { HolidayViewModel(holiday: $0)
                    }
                    self.holidays.onNext(holidayItems)
                    
                }) { (error) in
                    self.isLoading.onNext(false)
                    onError(error)
                }
            })
            .disposed(by: disposeBag)
    }
}
```
# Binding
```swift
// MARK: - Binding
extension HolidaysViewController {
    func bindTableView() {

        viewModel.holidays
            .bind(to: tableView.rx.items(cellIdentifier: "HolidayTableViewCell", cellType: HolidayTableViewCell.self)) { index, viewModel, cell in
                cell.viewModel = viewModel
            }
            .disposed(by: disposeBag)
        
        tableView.rx.modelSelected(HolidayViewModel.self)
            .bind(to: viewModel.selectedHoliday)
            .disposed(by: disposeBag)
        
        tableView.rx.itemSelected
            .subscribe(onNext: { (indexPath) in
                self.tableView.deselectRow(at: indexPath, animated: true)
            })
            .disposed(by: disposeBag)
    }
    
    func bindHUD() {
        
        viewModel.isLoading
            .subscribe(onNext: { [weak self] isLoading in
                isLoading ? self?.showProgress() : self?.hideProgress()
            })
            .disposed(by: disposeBag)
    }
    
    func bindNavItem() {
        chooseCountryItem.rx.tap
            .bind(to: viewModel.chooseCountry)
            .disposed(by: disposeBag)
        
        viewModel.selectedCountry
            .subscribe(onNext: { (country) in
                self.chooseCountryItem.title = country
            })
            .disposed(by: disposeBag)
    }
    
    func bindVisibilityState() {
        viewModel.selectedCountry
            .subscribe(onNext: { _ in
                self.tableView.isHidden = false
                self.chooseCountryLabel.isHidden = true
            })
            .disposed(by: disposeBag)
    }
}
```


```swift 
class ViewModel {
    private let disposeBag = DisposeBag()
    // MARK: - Actions
    let isLoading = BehaviorSubject<Bool>(value: false)
    func request() {
        isLoading.onNext(true)
        HolidaysService.shared.getHolidays(country: country, success: { (code, holidays) in
            self.isLoading.onNext(false)
        }
    }
}

class VC {
    func bindHUD() {
        viewModel.isLoading
            .subscribe(onNext: { [weak self] isLoading in
                isLoading ? self?.showProgress() : self?.hideProgress()
            })
            .disposed(by: disposeBag)
    }
}
```

```swift
import UIKit
import RxSwift
import RxDataSources
import RxCocoa

enum Cell {
    case title(String)
    case zalupa(String)
}

class ViewController: UIViewController {
    enum ActionCell {
        case tapCell(Int), showAlert
    }
    
    @IBOutlet var tableView: UITableView!
    @IBOutlet var switchView: UISwitch!
    
    let cells = BehaviorRelay<[Cell]>(value: [.title("ZalupaCell"), .zalupa("PupaCell"), .zalupa("PupaCell"), .zalupa("PupaCell")])
    let url = BehaviorRelay<String>(value: "")
    let bag = DisposeBag()
    let action = PublishRelay<ActionCell>()
    let getter = HTTPGet()
    let testBind = BehaviorRelay<Bool>(value: false)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        switchView.rx.value.bind(to: testBind).disposed(by: bag)
        
        testBind.bind { print($0) }.disposed(by: bag)
        
        cells.bind(to: tableView.rx.items) { table, row, data in
            switch data {
            case .zalupa(let title):
                let cell = table.dequeueReusableCell(withIdentifier: title)
                return cell!
            case .title(let title):
                let cell = table.dequeueReusableCell(withIdentifier: title)
                return cell!
            }
         
        }.disposed(by: bag)
        
        tableView.rx.itemSelected
            .map { ActionCell.tapCell($0.row) }
            .bind(to: action)
            .disposed(by: bag)
        
        action.bind {
            switch $0 {
            case .tapCell(let index):
                print("Hello \(index)")
            case .showAlert:
                print("Hello alert")
            }
        }.disposed(by: bag)
        
//        url.accept("https://rollstime.ru/api/catalog")
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [self] in
            getter.url.accept("https://jsonplaceholder.typicode.com/todos/2")

        }
        getter.url.accept("https://jsonplaceholder.typicode.com/todos/1")
        
        getter.data.bind { print($0) }.disposed(by: bag)
    }
    


}

class HTTPGet {
    let url = BehaviorRelay<String>(value: "")
    let data = BehaviorRelay<[String:Any]>(value: [:])
    private let bag = DisposeBag()

    init() {
        url.compactMap(URL.init)
            .map { URLRequest(url: $0) }
            .flatMap { URLSession.shared.rx.data(request: $0) }
            .compactMap { try? JSONSerialization.jsonObject(with: $0) as? [String:Any] }
            .bind { [weak self] in self?.data.accept($0) }
            .disposed(by: bag)
    }
}
```

```html
<dushnila> Hm >..< </dushnila>
<zanuda> gavno </zanuda>
<p> efef </p>
```
