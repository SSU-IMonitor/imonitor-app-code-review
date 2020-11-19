## LoginViewController

- ### loginViewModel = LoginViewModel()

  ```swift
  let idTextPublishSubject = PublishSubject<String>()
  let passwordTextPublishSubject = PublishSubject<String>()
  ```

  

  - #### PublishSubject<String>()

    - subject 구현에 필요한 onSubcribe로직 제공

    - subscribe를 subjectObserver로 포장하여 subscribers에 등록

      - ###### Subject<T, R>: observable<R>를 상속받고, observer를 구현하는 추상 클래스

    - onStart, onAdded, onTerminated등 호출

  

  </br>

  

  - #### Obeservable이란?

    - observable = observable sequence = sequence

    - 비동기적 성격

    - observable들은 일정 기간 동안 계속해서 이벤트를 생성(emitting 과정)

    - 각각의 이벤트들은 숫자나 커스텀한 인스턴스 등과 같은 값을 가질 수 있으며, 제스처 또한 인식이 가능하다.

    - ##### Observable의 생명주기

      - Observable은 어떤 구성요소를 가지는 next 이벤트를 계속해서 방출할 수 있다.
      - Observable은 error이벤트를 방출하여 완전 종료될 수 있다.
      - Observable은 complete 이벤트를 방출하여 완전 종료될 수 있다.

  

  </br>

  

  - #### 버튼 활성화

    > isValid함수는 idTextPublishSubject가 8이상이고, passwordPublishSubject가 1 이상일 때 observable에 false가 발생하여 Observable의 값을 true로 만들어준다.  

    ```swift
    func isValid() -> Observable<Bool> {
            Observable.combineLatest(idTextPublishSubject.asObservable().startWith(""), passwordTextPublishSubject.asObservable().startWith("")).map{
                id, password in
                return id.count >= 8 && password.count >= 1
            }.startWith(false)
        }
    ```

    - ##### combineLatest(): Observable의 데이터가 변경이되면, 그에 따라 observable들의 값을 합친 뒤 갱신하여 subscriber에 발행하는 함수이다.

      ![스크린샷 2020-11-18 오후 10.13.44](/Users/heoyeeun/Library/Application Support/typora-user-images/스크린샷 2020-11-18 오후 10.13.44.png)

    - ##### asObservable(): subject의 경우 Observable과 observer 두가지의 역할을 동시에 수행하는데, 이때 observable에 접근하지 못하도록 설정하여 observable에만 접근 가능하게 설정해주는 함수

    - ##### startsWith(""): 문자열의 첫번째 글자가 ""이면 true, 아니라면 false를 반환

  

  ######</br>

  

- ### disposeBag = DisposeBag()

  - observable은 subscribe 이후 complete 또는 error 이벤트가 발생하기 전까지 계속 next이벤트를 발생시키고, 이는 메모리 누수의 원인이 된다. 이때, Dispose는 메모리 관리를 도와주는 객체이다. (리소스와 실행 취소에 사용됨.)
  - disposeBag은 여러개의 subscribe를 한번에 해제할 수 있다.
  - subscribe가 반환하는 disposable이 disposBag에 담기고, disposeBag이 해제될 때 disposable이 모두 해제됨.
  - disposeBag이 해제되는 시점: bag = nil과 같이 직접 해제하거나, class가 deinit되면서 자동 해제



</br>



- ### setKeyboard()

  ```swift
  idTextField.addDoneButtonOnKeyboard()
  passwordTextField.addDoneButtonOnKeyboard()
  ```

  ```swift
  // addDoneButtonOnKeyboard: 키보드에 done 버튼을 추가해주는 역할
  extension UITextField{
      func addDoneButtonOnKeyboard(){
        // toolbar의 영역을 잡아줌.
          let doneToolbar: UIToolbar = UIToolbar(frame: CGRect.init(x: 0, y: 0, width: UIScreen.main.bounds.width, height: 50))
          doneToolbar.barStyle = .default
          
        	// toolbar에서 버튼의 위치를 설정
          let flexSpace = UIBarButtonItem(barButtonSystemItem: .flexibleSpace, target: nil, action: nil)
        
        // 버튼의 속성 설정 
        // #selector(): 메소드 이름을 인자값으로 넣어 전달하면 이를 내부적으로 정수값으로 매핑하여 처리하는 형태.
          let done: UIBarButtonItem = UIBarButtonItem(title: "Done", style: .done, target: self, action: #selector((self.doneButtonAction)))
          
        // toolbar를 display에 띄움
          let items = [flexSpace, done]
          doneToolbar.items = items
          doneToolbar.sizeToFit()
          
          self.inputAccessoryView = doneToolbar
      }
      
    	// resignFirstResponder: 텍스트 필드의 현재 상태를 포기했다는 요청을 receiver에게 알려주는 뜻. 이러한 형태로 활성화를 포기하면 키보드가 자동으로 내려가게 할 수 있다. 
      @objc func doneButtonAction(){
          self.resignFirstResponder()
      }
  }
  ```

  

<br>

- ### rxLogin()

  ```swift
  // becomeFirstResponder(): 특정뷰를 띄우면서 바로 특정 필드를 편집하고 싶을 때(= 키보드를 띄우고 싶은 위치에) 사용.
  idTextField.becomeFirstResponder()
  ```

  - ##### $0: 첫번 째 방출될(?) 값. 

    ex) print("\%0").disposed(by: disposeBag)을 두번 실행하면 값이 두번 방출됨.

  ```swift
  // signinButton이 눌릴 때 까지 텍스트에 대해  isValid() 함수를 호출하여 조건 체크
  // bind: 여러개가 동시에 하나를 참조
  loginViewModel.isValid().bind(to: signinButton.rx.isEnabled).disposed(by: disposeBag)
  
  // map: 이벤트를 바꾼다
  loginViewModel.isValid().map{ $0 ? 1 : 0.1}.bind(to: signinButton.rx.alpha).disposed(by: disposeBag)
  ```



</br>



- #### LoginInfo / User Info 생성

  ```swift
  struct LoginInfo: Codable{
      let accessToken: String
      let refreshToken: String
      let tokenType: String
      let userInfo: UserInfo
  }
  
  struct UserInfo: Codable{
      let id: String
      let major: String
      let name: String
  }
  ```

  - ##### Codable: Decodable과 Encodable 프로토콜을 준수하는 타입 

    - Decodable: 자신을 external representation에서 디코딩 할 수 있는 타입
    - Encodable: 자신을 external representation에서 인코딩 할 수 있는 타입

- ### 
