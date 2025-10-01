# PhoneBook

### Lv1.
> - `UILabel`, `UITableView`, `UIButton`을 이용해서 기본적인 UI를 구성합니다.
> - `UITableViewCell`에는 프로필 이미지를 보여줄 `UIImageView`와 이름을 보여줄 `UILabel`을 넣습니다. 전화번호를 표시할 `UILabel`도 넣습니다.
> - 프로필 이미지는 원모양이 되도록 합시다.
> - "추가" 버튼을 우상단에 위치 시킵니다.
> - Cell의 높이는 80으로 지정합니다.
> - 원의 테두리는 `layer.borderColor`, `layer.borderWidth`개념을 사용하면 구현할 수 있습니다.
> - 캡처된 화면은 임의의(더미) dataSource를 끼워넣어 UI를 확인한 모습입니다.


```
import UIKit

// 연락처를 표현할 데이터 모델
struct Contact: Codable {
    let name: String // 이름
    let phone: String // 전화번호
    let imageURL: String // 프로필 사진 URL (나중에 네트워크에서 이미지 불러올 것)
}

// UITableViewCell 커스텀 셀 클래스
class ContactCell: UITableViewCell {
    
    // 프로필 이미지 뷰 ( 원형 + 테두리 )
    let profileImageView: UIImageView = {
        let imageView = UIImageView()
        imageView.contentMode = .scaleAspectFill // 이미지 꽉 차게
        imageView.layer.cornerRadius = 30 // 원형 (반지름 30)
        imageView.layer.masksToBounds = true // 둥근 모서리 바깥은 잘라내기
        imageView.layer.borderColor = UIColor.gray.cgColor // 테두리 색
        imageView.layer.borderWidth = 1 // 테두리 두께
        return imageView
    }()
    
    // 이름 라벨
    let nameLabel: UILabel = {
        let label = UILabel()
        label.font = .systemFont(ofSize: 18, weight: .medium) // 18pt 중간 굵기 폰트
        return label
    }()
    
    // 전화번호 라벨
    let phoneLabel: UILabel = {
        let label = UILabel()
        label.font = .systemFont(ofSize: 14) // 14pt 폰트
        label.textColor = .darkGray // 회색 텍스트
        return label
    }()
    
    // 초기화 메서드 (셀UI 세팅)
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        // 셀의 contentView에 서브뷰 추가
        contentView.addSubview(profileImageView)
        contentView.addSubview(nameLabel)
        contentView.addSubview(phoneLabel)
        
        // 오토레이아웃 활성화
        profileImageView.translatesAutoresizingMaskIntoConstraints = false
        nameLabel.translatesAutoresizingMaskIntoConstraints = false
        phoneLabel.translatesAutoresizingMaskIntoConstraints = false
        
        
        // 오토레이아웃 제약 조건
        NSLayoutConstraint.activate([
            // 프로필 이미지는 왼쪽에 위치, 크기 60X60
            profileImageView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            profileImageView.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            profileImageView.widthAnchor.constraint(equalToConstant: 60),
            profileImageView.heightAnchor.constraint(equalToConstant: 60),
            
            // 이름 라벨은 이미지 오른쪽, 위쪽
            nameLabel.leadingAnchor.constraint(equalTo: profileImageView.trailingAnchor, constant: 16),
            nameLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 20),
            
            // 전화번호 라벨은 이름 라벨 바로 아래
            phoneLabel.leadingAnchor.constraint(equalTo: nameLabel.leadingAnchor),
            phoneLabel.topAnchor.constraint(equalTo: nameLabel.bottomAnchor, constant: 4)
        ])
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    // 셀에 데이터 넣기 (Contact 모델을 받아서 UI 업데이트)
    func configure(with contact: Contact) {
        nameLabel.text = contact.name // 이름 표시
        phoneLabel.text = contact.phone // 전화번호 표시
        
        // 프로필 사진 URL -> 이미지 뷰에 로드
        if let url = URL(string: contact.imageURL) {
            URLSession.shared.dataTask(with: url) { data, _, _ in
                if let data = data, let image = UIImage(data: data) {
                    // UI 변경은 반드시 메인 스레드에서 해야댐
                    DispatchQueue.main.async {
                        self.profileImageView.image = image
                    }
                }
                
            }.resume()
        }
    }
   
    }

// 연락처 리스트 화면 (테이블 뷰)
class ContactListViewController: UIViewController, UITableViewDataSource, UITableViewDelegate {
   
     private let tableView = UITableView()
    
    // 더미 연락처 데이터 (나중에 URL 넣어야댐)✨
    private var contacts: [Contact] = [
        Contact(name: "JK", phone: "010-0000-0901", imageURL: ""),
        Contact(name: "V", phone: "010-0000-1230", imageURL: ""),
        Contact(name: "JM", phone: "010-0000-1013", imageURL: "")
    ]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "연락처" // 네비게이션 바 타이틀
        view.backgroundColor = .white
        
        // 우측 상단 "추가" 버튼
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            title: "추가",
            style: .plain,
            target: self,
            action: #selector(addContact)
        )
        
        // 테이블 뷰 세팅
        tableView.dataSource = self
        tableView.delegate = self
        tableView.rowHeight = 80 // 셀 높이 고정
        tableView.register(ContactCell.self, forCellReuseIdentifier: "ContactCell") // 커스텀 셀 등록
        
        // 테이블 뷰를 화면에 추가
        view.addSubview(tableView)
        tableView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
    
    // "추가" 버튼 눌렀을 때 동작 (현재는 프린트만 구현)
    @objc private func addContact() {
        print("새 연락처 추가 예정")
    }
    
    // 테이블 뷰에 연락처 배열 개수만큼 보여주기
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return contacts.count
    }
    
    // 각 셀에 어떤 데이터를 보여줄지 설정
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // 커스텀 셀 가져오기
        guard let cell = tableView.dequeueReusableCell(withIdentifier: "ContactCell", for: indexPath) as? ContactCell else {
            return UITableViewCell()
        }
        // 해당 인덱스의 데이터로 셀 구성
        cell.configure(with: contacts[indexPath.row])
        return cell
    }
    <img width="499" height="945" alt="스크린샷 2025-09-30 19 40 32" src="https://github.com/user-attachments/assets/1e42db13-b8d4-4dd1-a894-9a736e7ce42b" />

}    

```
<img width="499" height="945" alt="스크린샷 2025-09-30 19 44 36" src="https://github.com/user-attachments/assets/ea888bb3-7ef5-4c62-ae30-6b51c6dfdf2b" />

오토레이아웃 부분은 스냅킷 설치 후 보기 좋게 다시 고치겠습니다.


### Lv 2~3

>**Lv2 - 연락처 추가 화면을 구현합니다**
>- 메인화면에서 "추가" 버튼을 누르면 이 페이지로 이동하도록 합니다. `UINavigationController.push`사용
>-  연락처를 편집하거나 새롭게 추가할 페이지를 개발합니다. (`PhoneBookViewController.swift`)
>- 프로필 이미지: `UIImageView`
>- 랜덤 이미지 생성 버튼: `UIButton`
>- 이름: `UITextView` 또는  `UITextField`
>- 전화번호: `UITextView` 또는 `UITextField`
>
>**Lv3 - 상단 네비게이션 바 영역을 구현합니다.**
>- 상단에 "제목"과 "적용"버튼을 개발합니다.
>- `UINavigationController`의 상단에는 `UINavigationBar`가 자동으로 생성됩니다.
>- `UINavigationItem`, `UINavigationBar`의 개념을 공부하면 개발할 수 있게 됩니다. 직접 공부하고 구현을 성공해봅시다.


```
import UIKit
import SnapKit

// 연락처 추가/ 편집 화면
class PhoneBookViewController: UIViewController {
    
    // 프로필 이미지 뷰
    private let profileImageView: UIImageView = {
        let imageView = UIImageView()
        imageView.contentMode = .scaleAspectFill // 이미지 비율 유지
        imageView.layer.cornerRadius = 50 // 원형
        imageView.layer.masksToBounds = true // 원형으로 잘라내기
        imageView.image = UIImage(systemName: "preson.circle.fill") // 기본이미지
        imageView.tintColor = .gray
        
        imageView.layer.borderColor = UIColor.darkGray.cgColor
        imageView.layer.borderWidth = 2
        return imageView
    }()
    
    // 랜덤 이미지 버튼
    private let randomImageButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("랜덤 이미지 생성", for: .normal) // 버튼 이름
        button.setTitleColor(.systemBlue, for: .normal) // 파란색 텍스트
        return button
    }()
    
    // 이름 입력 필드
    private let nameTextField: UITextField = {
        let textField = UITextField()
        textField.placeholder = "이름 입력" // 안내 문구
        textField.borderStyle = .roundedRect // 둥근 테두리
        return textField
    }()
    
    // 전화번호 입력 필드
    private let phoneTextField: UITextField = {
        let textField = UITextField()
        textField.placeholder = "전화번호 입력" // 안내 문구
        textField.borderStyle = .roundedRect
        textField.keyboardType = .phonePad // 숫자 키패드
        return textField
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        // 네비게이션 바 설정
        title = "새 연락처" // 화면 상단 네이게이션 타이틀
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            title: "적용", // 우측 버튼 이름
            style: .done, // 굵은 글씨 스타일
            target: self,
            action: #selector(applyContact) // 적용 버튼 눌렀을 때 실행될 함수
        )
        
        // UI 요소들 화면에 추가
        view.addSubview(profileImageView)
        view.addSubview(randomImageButton)
        view.addSubview(nameTextField)
        view.addSubview(phoneTextField)
        
        // 오토레이아웃 설정
        profileImageView.snp.makeConstraints {
            $0.top.equalTo(view.safeAreaLayoutGuide).offset(40)
            $0.centerX.equalToSuperview()
            $0.width.height.equalTo(100)
        }
        
        randomImageButton.snp.makeConstraints {
            $0.top.equalTo(profileImageView.snp.bottom).offset(12)
            $0.centerX.equalToSuperview()
        }
        
        nameTextField.snp.makeConstraints {
            $0.top.equalTo(randomImageButton.snp.bottom).offset(40)
            $0.leading.trailing.equalToSuperview().inset(20)
            $0.height.equalTo(44)
        }
        
        phoneTextField.snp.makeConstraints {
            $0.top.equalTo(nameTextField.snp.bottom).offset(20)
            $0.leading.trailing.equalTo(nameTextField)
            $0.height.equalTo(44)
        }
        
        // 랜덤 이미지 버튼을 눌렀을 때 실행될 함수 연결
        randomImageButton.addTarget(self, action: #selector(loadRandomImage), for: .touchUpInside)
    }
    
    // "적용" 버튼 눌렀을 때
    @objc private func applyContact() {
        
        //사용자가 입력한 값 가져오기
        let name = nameTextField.text ?? ""
        let phone = phoneTextField.text ?? ""
        
        // 나중에 수정해야댐✨
        print("적용 버튼 눌림 -> 이름: \(name), 전화번호: \(phone)")
        
        // 현재 화면 닫고 이전 화면으로 돌아가기
        navigationController?.popViewController(animated: true)
    }
    
    // 랜덤 이미지 불러오기
    @objc private func loadRandomImage() {
        // 나중에 추가✨
         let url = URL(string: "")!
        
        // 네트워크에서 이미지 데이터 가져오기
        URLSession.shared.dataTask(with: url) { data, _, _ in
            if let data = data, let image = UIImage(data: data) {
                
                // UI 변경은 메인 스레드에서!
                DispatchQueue.main.async {
                    self.profileImageView.image = image
                }
            }
            
        }.resume() // 네트워크 요청 시작
    }
}

```

<img width="455" height="901" alt="스크린샷 2025-10-01 20 51 57" src="https://github.com/user-attachments/assets/f471ca28-f273-465e-aa65-00dfca360cd4" />



처음에 Lv1에서 구현했던 "추가" 버튼 동작 프린트 부분을 고쳐줘야 "추가" 버튼 작동함

```
// "추가" 버튼 눌렀을 때 동작
    @objc private func addContact() {
        
        let vc = PhoneBookViewController()
        
        navigationController?.pushViewController(vc, animated: true)
    }
```
