# TIL_jdbc

package config;

public interface ServerInfo {
	String DRIVER="com.mysql.cj.jdbc.Driver";//public static final 무조건 생략
	String URL = "jdbc:mysql://127.0.0.1:3306/scott?serverTimezone=UTC&useUnicode=yes&characterEncoding=UTF-8";
	String USER= "root";
	String PASSWORD = "1234";
}
----------------------------------------------------------------
package com.encore.vo;
/*
 * db테이블의 Record를 인스턴스화 시킨 클래스 
 * DO(Domain)---> VO(Value Object) --->DTO(Domain Transfer Object)
 * 테이블의 컬럼이 클래스의 필드와 연결된다.
 * 
 */
public class Person {

	private int ssn;
	private String name;
	private String address;
	
	public Person() {
		
	}

	public Person(int ssn, String name, String address) {
		super();
		this.ssn = ssn;
		this.name = name;
		this.address = address;
	}
	
	public int getSsn() {
		return ssn;
	}
	public void setSsn(int ssn) {
		this.ssn = ssn;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getAddress() {
		return address;
	}
	public void setAddress(String address) {
		this.address = address;
	}
	@Override
	public String toString() {
		return "Person [ssn=" + ssn + ", name=" + name + ", address=" + address + "]";
	}
	
	
	
}




--------------------------------------
package com.encore.test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;

import com.encore.vo.Person;

import config.ServerInfo;

/*
 * DAO + Test
 * JDBC 4단계(드라이버 로딩은 제외)
 * 2. 디비연결
 * 3. PreparedStatement 생성
 * 4. 쿼리문 실행 및 바인딩
 * 5. 자원 닫기
 * ---> 이중에서 
 * 메소드마다 고정적으로 바뀌지 않고 로직이 작성되는 부분은 2와 5번이다.
 * 이런 부분은 위에다 메소드를 새롭게 정의하고 
 * 메소드마다 호출해서 재사용하는 방법으로 로직을 다시 작성해야 한다.
 * 
 */
public class SimplePersonJDBCTest {
	
	public SimplePersonJDBCTest() throws ClassNotFoundException{
		
		Class.forName(ServerInfo.DRIVER);
		System.out.println("Driver Loading...");
	}
	
	//고정적으로 반복되는 부분을 공통적인 로직으로 정의...getConnection(), close()
	public Connection getConnection() throws SQLException{
		Connection conn = DriverManager.getConnection(ServerInfo.URL, ServerInfo.USER, ServerInfo.PASSWORD);
		System.out.println("디비 서버 연결");
		return conn;
	}
	
	public void closeAll(PreparedStatement ps, Connection conn) throws SQLException{
		if(ps!=null) ps.close();
		if(conn!=null) conn.close();
	}
	
	public void closeAll(ResultSet rs, PreparedStatement ps, Connection conn) throws SQLException{
		if(rs!=null) rs.close();
		closeAll(ps, conn);
	}
	
	// 비지니스로직 정의(DB Access하는 로직) 정의
	public void addPerson(Person p) throws SQLException {
		Connection conn = null;
		PreparedStatement ps = null;
		//2단계
		conn = getConnection();
		
		//3단계
		String query = "INSERT INTO person VALUES(?,?,?)";
		ps = conn.prepareStatement(query);
		System.out.println("PreparedStatement 생성");
		
		//4단계
		ps.setInt(1, p.getSsn());
		ps.setString(2, p.getName());
		ps.setString(3, p.getAddress());
		System.out.println(ps.executeUpdate()+"명 추가되었습니다.");
		
		//5단계 (중요)
		closeAll(ps, conn);
	}
	
	public void deletePerson(int ssn) throws SQLException{
		Connection conn = null;
		PreparedStatement ps = null;
		//2
		conn = getConnection();
		
		//3단계
		String query = "DELETE FROM person WHERE ssn =?";
		ps = conn.prepareStatement(query);
		System.out.println("PreparedStatement 생성");
		
		//4단계
		ps.setInt(1, ssn);
		System.out.println(ps.executeUpdate()+"명 삭제되었습니다.");
		
		//5단계 (중요)
		closeAll(ps, conn);
	}
	
	public void updatePerson(Person p) throws SQLException{
		Connection conn = null;
		PreparedStatement ps = null;
		//2
		conn = getConnection();
		//3
		String query = "UPDATE person set name=?, address=? WHERE ssn=?";
		ps = conn.prepareStatement(query);
		System.out.println("PreparedStatement 생성");
		//4
		ps.setString(1, p.getName());
		ps.setString(2, p.getAddress());
		ps.setInt(3, p.getSsn());
		System.out.println(ps.executeUpdate()+"명 정보가 수정되었습니다.");
		//5
		closeAll(ps, conn);
	}
	
	//select..conn, ps, rs...close?
	public ArrayList<Person> findAllPerson() throws SQLException{
		Connection conn = null;
		PreparedStatement ps = null;
		ResultSet rs = null;
		
		ArrayList<Person> list = new ArrayList<Person>();
		//2
		conn = getConnection();
		//3
		String query = "SELECT ssn, name, address From person";
		ps = conn.prepareStatement(query);
		System.out.println("PreparedStatement 객체 생성");
		//4
		rs = ps.executeQuery();
		
		while(rs.next()) {
				list.add(new Person(rs.getInt("ssn"),
									rs.getString("name"),
									rs.getString("address")));		
		}
		closeAll(rs, ps, conn);
					return list;
	}
	
	public static void main(String[] args)throws ClassNotFoundException, Exception {
		SimplePersonJDBCTest jdbc = new SimplePersonJDBCTest ();
		//jdbc.addPerson(new Person(333, "Tomas", "Canada"));
		//jdbc.deletePerson(222);
		//jdbc.updatePerson(new Person(333,"Tomas", "Brandon"));
		//ArrayList<Person> returnList = jdbc.findAllPerson();
		//for(Person p : returnList) System.out.println(p);
	}
		
}
