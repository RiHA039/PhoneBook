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
<img width="499" height="945" alt="스크린샷 2025-09-30 19 40 32" src="https://github.com/user-attachments/assets/61f9c659-53ae-4d73-982f-6f9981b09623" />

