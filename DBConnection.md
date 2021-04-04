# TIL_jdbc

package jdbc.test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/*
 * 1. 서버와 연결하기 위해서 먼저
 *    Driver를 메모리에 로딩
 * 2. 디비 서버와 연결하기
 * 	  연결에 성공하면 Connection 객체가 반환된다.
 * 3. PreparedStatement륿 반환받는다.
 * 		2번에서 생성된 connection이 그 기능을 가지고 있다.
 * 4. 3번에서 생성된 PreparedStatement의 기능..executeQuery()를 이용해서
 * 	  SQL문을 실행
 */
public class DBConnectionTest1 {

	public DBConnectionTest1() {
		
		try {
			//1.드라이버 로딩
			Class.forName("com.mysql.cj.jdbc.Driver");
			
			System.out.println("Driver Loading Success!!");
			
			//2. 디비서버와 연결
			String url ="jdbc:mysql://127.0.0.1:3306/scott?serverTimezone=UTC&useUnicode=yes&characterEncoding=UTF-8";
			Connection conn = DriverManager.getConnection(url,"root","1234");
			System.out.println("DB Server Connection...OK!");
			
			//3.PreparedStatement 생성
			String sql = "SELECT*FROM myscott";
			PreparedStatement ps = conn.prepareStatement(sql);
			System.out.println("PreparedStatement...Creating!");
			
			//4.SQL문 실행...
			ResultSet rs = ps.executeQuery();
			while(rs.next()){
				System.out.println(rs.getString(1)+","+rs.getInt(2));
			}
		}catch(ClassNotFoundException e){
			System.out.println("Driver Loading Fail!!");
		}catch(SQLException e) {
			System.out.println("DB Driver Connection.... Fail!!");
		}
	}

	public static void main(String[] args) {
		
		new DBConnectionTest1();
	}

}
---------------------------------------------------------

package jdbc.test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;

/*
 * 디비 서버 정보에 해당하는 것들을 상수값으로 세팅
 * 1. 드라이버 FQCN---- 이 부분이 먼저 메모리에 로딩된 다음에 일들이 일어나야 한다.
 * 2. 서버주소, 계정이름, 비번...
 * 
 */
public class DBConnectionTest2 {
	public static String DRIVER="com.mysql.cj.jdbc.Driver";
	public static String URL = "jdbc:mysql://127.0.0.1:3306/scott?serverTimezone=UTC&useUnicode=yes&characterEncoding=UTF-8";
	public static String USER= "root";
	public static String PASSWORD = "1234";
	
	public DBConnectionTest2() {
		//jdbc 기본작업 진행..2,3,4 단계 작업을 진행한다.
		try {
			//2
			Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
			System.out.println("디비서버 연결 성공! ");
			
			//3.
//			String insertQuery = "INSERT INTO mytable(num, name, address) VALUES(?,?,?)";
//			PreparedStatement ps = conn.prepareStatement(insertQuery);
//			System.out.println("PreparedStatement 생성! ");
//			
//			//4. 값 바인딩 및 쿼리문 실행
//			ps.setInt(1, 111);
//			ps.setString(2, "박나래");
//			ps.setString(3, "여의도");
//			
//			int row=ps.executeUpdate();// 업데이트된(추가) row의 객수가 반환
//			System.out.println(row+"명 추가됨!!");
			
			String deleteQuery = "DELETE FROM mytable WHERE name=?";
			PreparedStatement ps2 = conn.prepareStatement(deleteQuery);
			ps2.setString(1,"박나래");
			System.out.println(ps2.executeUpdate()+"명 삭제됨!!");
		}catch(Exception e) {
			System.out.println("디비서버 연결 실패!");
		}
	}
	
	public static void main(String[] args) {
		new DBConnectionTest2();
	}
	static {
		//1. 드라이버 로딩 작업
		try {
			Class.forName(DRIVER);
			System.out.println("Driver 로딩 성공! ");
		} catch (ClassNotFoundException e) {
			System.out.println("Driver 로딩 실패! ");
		}
	}
}

