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
