# TIL_jdbc

package config;

public interface ServerInfo {
	String DRIVER="com.mysql.cj.jdbc.Driver";//public static final 무조건 생략
	String URL = "jdbc:mysql://127.0.0.1:3306/scott?serverTimezone=UTC&useUnicode=yes&characterEncoding=UTF-8";
	String USER= "root";
	String PASSWORD = "1234";
}

--------------------------------------------------------

package broker.twotier.exception;

public class DuplicateSSNException extends Exception{

	public DuplicateSSNException() {
		this("SSN이 중복되었습니다!");
	}

	public DuplicateSSNException(String message) {
		super(message);
	}
}

--------------------------------------------------------

package broker.twotier.exception;

public class InvalidTransactionException extends Exception{
	public InvalidTransactionException() {
		this("거래가 불가능합니다.");
	}

	public InvalidTransactionException(String message) {
		super(message);
	}
}
-----------------------------------------------------
package broker.twotier.exception;

public class RecordNotFoundException extends Exception{
	public RecordNotFoundException() {
		this("데이터를 찾을 수 없습니다.");
	}

	public RecordNotFoundException(String message) {
		super(message);
	}
}
---------------------------------------------------

package broker.twotier.vo;

import java.util.Vector;

/**
 * 
 * @author Sujin
 * 
 * 고객에 대한 정보를 저장하는 Record Class
 * 해당 고객은 주식을 사거나 파는데 연관이 있는 고객
 * 
 * 주식을 보유한 고객 | 주식을 보유하지 않은 고객일 수 있다.
 *
 */
public class CustomerRec {
	private String ssn;
	private String name; // 컬럼명은 cust_name
	private String address;
	
	//필드추가
	private Vector<SharesRec> portfolio;
	
	public CustomerRec(String ssn, String name, String address, Vector<SharesRec> portfolio) {
		super();
		this.ssn = ssn;
		this.name = name;
		this.address = address;
		this.portfolio = portfolio;
	}

	public CustomerRec(String ssn, String name, String address) {
		super();
		this.ssn = ssn;
		this.name = name;
		this.address = address;
	}

	public CustomerRec() {
		
	}

	public String getSsn() {
		return ssn;
	}

	public void setSsn(String ssn) {
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

	public Vector<SharesRec> getPortfolio() {
		return portfolio;
	}

	public void setPortfolio(Vector<SharesRec> portfolio) {
		this.portfolio = portfolio;
	}

	@Override
	public String toString() {
		return "CustomerRec [ssn=" + ssn + ", name=" + name + ", address=" + address + ", portfolio=" + portfolio + "]";
	}
}
-----------------------------------------------------------

package broker.twotier.vo;

/**
 * 
 * @author Sujin
 * 
 * 누가 어떤 주식을 
 * 얼마만큼 보유하고 있는지의 정보를 담고 있는 클래스.. 
 * 
 * shares table의 정보
 *
 */
public class SharesRec {

	private String ssn;
	private String symbol;
	private int quantity;
	
	public SharesRec(String ssn, String symbol, int quantity) {
		super();
		this.ssn = ssn;
		this.symbol = symbol;
		this.quantity = quantity;
	}
	public SharesRec() {
		super();
		// TODO Auto-generated constructor stub
	}
	public String getSsn() {
		return ssn;
	}
	public void setSsn(String ssn) {
		this.ssn = ssn;
	}
	public String getSymbol() {
		return symbol;
	}
	public void setSymbol(String symbol) {
		this.symbol = symbol;
	}
	public int getQuantity() {
		return quantity;
	}
	public void setQuantity(int quantity) {
		this.quantity = quantity;
	}
	@Override
	public String toString() {
		return "SharesRec [ssn=" + ssn + ", symbol=" + symbol + ", quantity=" + quantity + "]";
	}
	
	
}
-----------------------------------------------------------

package broker.twotier.vo;
/**
 * 
 * @author Sujin
 * 
 * Stock 테이블의 정보를 저장하는 클래스...
 * 주식의 이름, 주식의 가격으로 필드가 구성된다.
 *
 */
public class StockRec {
	
	private String symbol;
	private float price;
	
	public StockRec(String symbol, float price) {
		
		this.symbol = symbol;
		this.price = price;
	}
	public StockRec() {
		this("",0.0F);
	}
	public String getSymbol() {
		return symbol;
	}
	public void setSymbol(String symbol) {
		this.symbol = symbol;
	}
	public float getPrice() {
		return price;
	}
	public void setPrice(float price) {
		this.price = price;
	}
	@Override
	public String toString() {
		return "StockRec [symbol=" + symbol + ", price=" + price + "]";
	}
	
	
	
}


-------------------------------------------------------
package broker.twotier.dao;


import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Vector;
import broker.twotier.exception.DuplicateSSNException;
import broker.twotier.exception.InvalidTransactionException;
import broker.twotier.exception.RecordNotFoundException;
import broker.twotier.vo.CustomerRec;
import broker.twotier.vo.SharesRec;
import broker.twotier.vo.StockRec;
import config.ServerInfo;

public class Database implements DatabaseTemplate{

	public Database(String serverIp)throws ClassNotFoundException{
		Class.forName(ServerInfo.DRIVER);
		System.out.println("Driver Loading Success");
	}
	
	@Override
	public Connection getConnect() throws SQLException {
		Connection conn = DriverManager.getConnection(ServerInfo.URL, ServerInfo.USER, ServerInfo.PASSWORD);
		System.out.println("디비 서버 연결");
		return conn;	
	}

	@Override
	public void closeAll(PreparedStatement ps, Connection conn) throws SQLException {
		
		if(ps!=null) ps.close();
		if(conn!=null) conn.close();
		
	}

	@Override
	public void closeAll(ResultSet rs, PreparedStatement ps, Connection conn) throws SQLException {
	
		if(rs!=null) rs.close();
		closeAll(ps, conn);
	}

	//ssn이 있는지 없는지 확인하는 기능.
	public boolean isSsn(String ssn, Connection conn) throws SQLException{
		
		String query = "SELECT ssn FROM customer WHERE ssn=?";
		PreparedStatement ps = conn.prepareStatement(query);
		ps.setString(1, ssn);
		ResultSet rs = ps.executeQuery();
		return rs.next(); // true, false
		
	}
	
	@Override
	public void addCustomer(CustomerRec cust) throws SQLException, DuplicateSSNException {
		Connection conn = null;
		PreparedStatement ps = null;
		
		//2단계
		try{
			conn = getConnect();//
			
			if(!isSsn(cust.getSsn(),conn)) { 
				System.out.println("회원등록 가능");
				String query = "INSERT INTO Customer(ssn, cust_name, address) VALUES(?,?,?)";
				ps = conn.prepareStatement(query);
				
				ps.setString(1, cust.getSsn());
				ps.setString(2, cust.getName());
				ps.setString(3, cust.getAddress());
				
				System.out.println(ps.executeUpdate()+" 명 INSERT OK ");
			}else { 
				throw new DuplicateSSNException();
			}
		}finally {
			closeAll(ps, conn);
		}
		
	}

	@Override
	public void deleteCustomer(String ssn) throws SQLException, RecordNotFoundException {
		Connection conn = null;
		PreparedStatement ps = null;
		PreparedStatement ps1 = null;
		
		 try {
			 conn = getConnect();
			 if(!isSsn(ssn, conn)) { 
				System.out.println("회원삭제 가능");
				
				String query = "DELETE FROM customer WHERE ssn = ?";
				ps = conn.prepareStatement(query);
			 
				String query1 = "DELETE FROM shares WHERE ssn = ?";
				ps1 = conn.prepareStatement(query1);
				
				ps.setString(1, ssn);
				ps1.setString(1, ssn);
				
				System.out.println(ps.executeUpdate()+" 명 Delete!!");
				System.out.println(ps1.executeUpdate()+" 명 Delete!!...shares");
			 }else {
				 throw new RecordNotFoundException();
			 }
		 }finally {
			 closeAll(ps, conn);
		 }
	}

	@Override
	public void updateCustomer(CustomerRec cust) throws SQLException, RecordNotFoundException {
		
		Connection conn = null;
		PreparedStatement ps = null;
		
		//2단계
		try{
			conn = getConnect();//
			
			if(!isSsn(cust.getSsn(),conn)) { 
				System.out.println("수정 가능");
				String query = "UPDATE customer set name=?, address=? WHERE ssn=?";
				
				ps = conn.prepareStatement(query);
				
				ps.setString(1, cust.getName());
				ps.setString(2, cust.getAddress());
				ps.setString(3, cust.getSsn());
				//ps.setVector<SharesRec>(4, cust.getPortfolio(new SharesRec()));
				int row = ps.executeUpdate();
				//if(row==1) 
					System.out.println(ps.executeUpdate()+" 명 UPDATE OK!");
			}else { 
				throw new RecordNotFoundException();
			}
		}finally {
			closeAll(ps, conn);
		}
	}

	@Override
	public Vector<SharesRec> getPortfolio(String ssn) throws SQLException, RecordNotFoundException {
		Connection conn = null;
		PreparedStatement ps = null;
		ResultSet rs = null;
		Vector<SharesRec> sr = new Vector<SharesRec>();
		//2단계
		try{
			conn = getConnect();//
			
			if(!isSsn(ssn,conn)) { 
				System.out.println("회원 확인 완료!");
				String query = "SELECT ssn, symbol, quantity from Shares WHERE ssn=?";
				ps = conn.prepareStatement(query);
				
				ps.setString(1, ssn);
				
				rs = ps.executeQuery();
				
				while(rs.next()) {
					sr.add(new SharesRec(ssn, 
										rs.getString("symbol"),
										rs.getInt("quantity")));
				}
				
			}else { 
				throw new RecordNotFoundException();
			}
		}finally {
			closeAll(ps, conn);
		}
		return sr;
	}

	@Override
	public CustomerRec getCustomer(String ssn) throws SQLException, RecordNotFoundException{
		Connection conn = null;
		PreparedStatement ps = null;
		ResultSet rs = null;
		CustomerRec cust = null;
		try {
			conn=getConnect();
			String query = "SELECT ssn, cust_name, address from Customer where ssn=?";
			ps = conn.prepareStatement(query);
			
			ps.setString(1, ssn);
			
			rs = ps.executeQuery();
			
			if(rs.next()) { //ssn에 해당하는 고객이 있다면
				cust = new CustomerRec(ssn, 
									rs.getString("cust_name"), 
									rs.getString("address"));
			}
			cust.setPortfolio(getPortfolio(ssn));
		}finally {
			closeAll(rs, ps, conn);
		}
		return cust;
	}

	@Override
	public ArrayList<CustomerRec> getAllCustomers() throws SQLException, RecordNotFoundException {
		Connection conn = null;
		PreparedStatement ps = null;
		ResultSet rs = null;
		ArrayList<CustomerRec> list = new ArrayList<>();
		try {
			conn=getConnect();
			String query = "SELECT ssn, cust_name, address from Customer where ssn=?";
			ps = conn.prepareStatement(query);
			
			rs = ps.executeQuery();
			
			while(rs.next()) {
				list.add(new CustomerRec(
						rs.getString("ssn"),
						rs.getString("cust_name"),
						rs.getString("address"),
						getPortfolio(rs.getString("ssn"))));
		}
		}finally {
			closeAll(rs, ps, conn);
		}
		return list;
	}

	
	
	@Override
	public ArrayList<StockRec> getAllStocks() throws SQLException {
		
		return null;
	}
/*
 * 누가 어떤 주식을 몇개 살지를 정희하는 기능
 * 
 * 지금 가지고 있는 주식의 갯수(quantity)부터 확인해봐야 한다.
 * 
 * 내가 현재 주식을 안가지고 있다 ---> insert into
 * 내가 현재 어느정도의 주식을 가지고 있다. --> update
 */
	@Override
	public void buyShares(String ssn, String symbol, int quantity) throws SQLException {
		Connection conn = null;
		PreparedStatement ps = null;
		ResultSet rs = null;
		try {
			conn=getConnect();
			String query = "SELECT quantity from shares where ssn=? and symbol= ?";
			ps = conn.prepareStatement(query);
			ps.setString(1, ssn);
			ps.setString(2, symbol);
			
			rs=ps.executeQuery();
			
			if(rs.next()) {
				int q=rs.getInt(1); // q는 현재 가지고 있는 주식의 수량
				int newQuantity = q + quantity; //q(0) + quantity(100) | q(50) + quantity(100)
				//update
				String query1 = "UPDATE shares set quantity=? where ssn = ? and symbol =?";
				ps = conn.prepareStatement(query1);
				ps.setInt(1, newQuantity);
				ps.setString(2, ssn);
				ps.setString(3, symbol);
				
				System.out.println(ps.executeUpdate()+" row buyShares...");
			}else {
				//insert
				String query2 = "INSERT INTO shares (ssn, symbol, quantity) values(?,?,?)";
				ps = conn.prepareStatement(query2);
				ps.setString(1, ssn);
				ps.setString(2, symbol);
				ps.setInt(3, quantity);
				
				System.out.println(ps.executeUpdate()+" row buyShares...");
			}
		}finally {
			closeAll(rs, ps, conn);
		}
		
	}
/*
 * 누가 어떤 주식을 몇개 팔 것인가에 대한 기능을 정의....
 * 현재 가지고 있는 주식의 수량을 먼저 알아야한다. int q에 할당
 * 
 * 1) 100개를 가지고 있다 --- 50 sell ----update
 * 2) 100개를 가지고 있다 --- 100 sell ----delete
 * 3) 100개를 가지고 있다 --- 200 sell ---- InvalidTransactionException
 * 4) 0개를 가지고 있다 ---------------------RecordNotFoundException
 */
	@Override
	public void sellShares(String ssn, String symbol, int quantity)
			throws SQLException, InvalidTransactionException, RecordNotFoundException {
//		
		Connection conn = null;
		 PreparedStatement ps = null;	
		 ResultSet rs = null;
		 try {
			 conn=  getConnect();
			 
			 String query ="SELECT quantity FROM shares WHERE ssn=? AND symbol=?";
			 ps = conn.prepareStatement(query);
			 ps.setString(1, ssn);
			 ps.setString(2, symbol);
			 
			 rs = ps.executeQuery();
			 
			 rs.next();//일단 커서를 한단계 아래로 내려서 엘러먼트를 가리키게 하고 수량을 받아올 준비를 한다.
			 
			 int q = rs.getInt(1); // 현재 가지고 있는 수량...100
			 int newQuantity = q-quantity; //팔고 남은 수량
			 
			 if(q==quantity) { //delete
				 String query1 = "DELETE FROM shares WHERE ssn=? AND symbol=?";
				 ps = conn.prepareStatement(query1);
				 ps.setString(1, ssn);
				 ps.setString(2, symbol);
				 
				 System.out.println(ps.executeUpdate()+" row SHARES DELETE....sellShares()1...");
			 }else if(q>quantity) { //update
				 String query2 = "UPDATE shares SET quantity=? WHERE ssn=? AND symbol=?";
				 ps = conn.prepareStatement(query2);
				 
				 ps.setInt(1, newQuantity);
				 ps.setString(2, ssn);
				 ps.setString(3, symbol);
				 
				 System.out.println(ps.executeUpdate()+" row SHARES UPDATE....sellShares()2...");
			 }else {  //펑
				 throw new InvalidTransactionException();
			 }		 			 
		 }finally {
			 closeAll(rs, ps, conn);
		 }
	}

}

--------------------------------------------------------------------------------

package broker.twotier.dao;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Vector;

import broker.twotier.exception.DuplicateSSNException;
import broker.twotier.exception.InvalidTransactionException;
import broker.twotier.exception.RecordNotFoundException;
import broker.twotier.vo.CustomerRec;
import broker.twotier.vo.SharesRec;
import broker.twotier.vo.StockRec;
import config.ServerInfo;
/*
 * 싱글톤 사용 X
 */
public class DatabaseSample implements DatabaseTemplate{
	
	public DatabaseSample(String serverIp) throws ClassNotFoundException{
		Class.forName(ServerInfo.DRIVER);
		System.out.println("드라이버 로딩 성공..");
	}
	@Override
	public Connection getConnect() throws SQLException {
		Connection conn =DriverManager.getConnection(ServerInfo.URL, ServerInfo.USER, ServerInfo.PASSWORD);
		System.out.println("Database Connection......");
		return conn;
	}

	@Override
	public void closeAll(PreparedStatement ps, Connection conn) throws SQLException {
		if(ps!=null) ps.close();
		if(conn!=null) conn.close();			
	}

	@Override
	public void closeAll(ResultSet rs, PreparedStatement ps, Connection conn) throws SQLException {
		if(rs!=null) rs.close();
		closeAll(ps, conn);			
	}
	
	//ssn이 있는지 없는지 확인하는 기능....하나 추가...isIsbn()과 동일한 기능
	public boolean isExist(String ssn, Connection conn)throws SQLException{
		String sql ="SELECT ssn FROM customer WHERE ssn=?";
		PreparedStatement ps = conn.prepareStatement(sql);
		
		ps.setString(1,ssn);
		ResultSet rs = ps.executeQuery();
		return rs.next();
	}

	@Override
	public void addCustomer(CustomerRec cust) throws SQLException, DuplicateSSNException {
		Connection conn = null;
		PreparedStatement ps = null;
		try {
			conn = getConnect();
			
			if(!isExist(cust.getSsn(), conn)) { //추가하려는 고객의 ssn이 없다면...추가
				String query = "INSERT INTO customer (ssn, cust_name, address) VALUES(?,?,?)";
				ps = conn.prepareStatement(query);
				
				ps.setString(1, cust.getSsn());
				ps.setString(2, cust.getName());
				ps.setString(3, cust.getAddress());
				
				System.out.println(ps.executeUpdate()+" 명 INSERT OK....addCustomer() ..");
			}else {
				throw new DuplicateSSNException();
			}			
		}finally {
			closeAll(ps, conn);
		}		
	}
//
	@Override
	public void deleteCustomer(String ssn) throws SQLException, RecordNotFoundException {
		Connection conn = null;
		PreparedStatement ps = null;
		//외래키 지정을 제약조건에 추가.. customer 테이블에서 삭제를 하게되면 연결된 ssn이 shares 테이블에서도 자동 삭제될 것이다.
		try {
			conn = getConnect();
			if(isExist(ssn, conn)) {
				String query = "DELETE FROM customer WHERE ssn=?";
				ps = conn.prepareStatement(query);
			
				ps.setString(1, ssn);

				System.out.println(ps.executeUpdate()+" 명 DELETE OK...deleteCustomer()...");

			}else {
				throw new RecordNotFoundException();
			}			
		}finally {
			closeAll(ps, conn);
		}		
	}

	@Override
	public void updateCustomer(CustomerRec cust) throws SQLException, RecordNotFoundException {
		Connection conn = null;
		PreparedStatement ps = null;
		try {
			conn = getConnect();
			String query = "UPDATE customer SET cust_name=?, address = ?  WHERE ssn=?";
			
			ps = conn.prepareStatement(query);
			ps.setString(1, cust.getName());
			ps.setString(2, cust.getAddress());
			ps.setString(3, cust.getSsn());
			
			int row = ps.executeUpdate();
			if(row==1) System.out.println(row+" 명 UPDATE OK...updateCustomer()..");
			else throw new RecordNotFoundException();
		}finally {
			closeAll(ps, conn);
		}		
	}

	@Override
	public Vector<SharesRec> getPortfolio(String ssn) throws SQLException {
		 Connection conn = null;
		 PreparedStatement ps = null;	
		 ResultSet rs = null;
		 Vector<SharesRec> v = new Vector<SharesRec>();
		 try{
			 conn = getConnect();
			 
			 String query ="SELECT ssn, symbol, quantity FROM shares WHERE ssn=?";
			 ps = conn.prepareStatement(query);
			 
			 ps.setString(1, ssn);
			 rs = ps.executeQuery();
			 while(rs.next()) {
				 v.add(new SharesRec(ssn, 
						 			 rs.getString("symbol"), 
						 			 rs.getInt("quantity")));
			 }
		 }finally {
			 closeAll(rs, ps, conn);
		 }
		
		return v;
	}

	@Override
	public CustomerRec getCustomer(String ssn) throws SQLException {
		Connection conn = null;
		PreparedStatement ps = null;	
		ResultSet rs = null;
		CustomerRec cust = null;
		try {
			conn = getConnect();
			String query = "SELECT ssn, cust_name, address FROM customer WHERE ssn=?";
			ps = conn.prepareStatement(query);
			ps.setString(1, ssn);
			
			rs = ps.executeQuery();
			if(rs.next()) { //ssn에 해당하는 고객이 있다면
				cust = new CustomerRec(ssn, 
									   rs.getString("cust_name"),
									   rs.getString("address"));
			}//if
			cust.setPortfolio(getPortfolio(ssn));
			
		}finally {
			closeAll(rs, ps, conn);
		}
		return cust;
	}

	@Override
	public ArrayList<CustomerRec> getAllCustomers() throws SQLException {
		Connection conn = null;
		PreparedStatement ps = null;	
		ResultSet rs = null;
		ArrayList<CustomerRec> list = new ArrayList<>();
		try {
			conn = getConnect();
			
			String query = "SELECT ssn, cust_name, address FROM customer";
			ps = conn.prepareStatement(query);
			
			rs = ps.executeQuery();
			while(rs.next()) {
				list.add(new CustomerRec(
							rs.getString("ssn"), 
							rs.getString("cust_name"), 
							rs.getString("address"),
							getPortfolio(rs.getString("ssn"))));
			}
		}finally {
			closeAll(rs, ps, conn);
		}
		return list;
	}

	@Override
	public ArrayList<StockRec> getAllStocks() throws SQLException {
		 Connection conn = null;
		 PreparedStatement ps = null;	
		 ResultSet rs = null;
		 ArrayList<StockRec> list = new ArrayList<StockRec>();
		 try{
			 conn = getConnect();
			 String query = "SELECT symbol, price FROM stock";
			 ps = conn.prepareStatement(query);
			 rs = ps.executeQuery();
			 while(rs.next()){
				 list.add(new StockRec(rs.getString(1), 
						 			   rs.getFloat(2)));
			 }
		 }finally{
			 closeAll(rs, ps, conn);
		 }
		 return list;
	}
/*
 * 누가 어떤 주식을 몇개 살지를 정의하는 기능...
 * 
 * 지금 가지고 있는 주식의 갯수(quantity)부터 확인해봐야 한다.
 * 
 * 내가 현재 주식을 안가지고 있다 0 , 100--> insert into  100
 * 내가 현재 어느정도의 주식을 가지고 있다 50, 100--> update 150
 */
	@Override
	public void buyShares(String ssn, String symbol, int quantity) throws SQLException {
		 Connection conn = null;
		 PreparedStatement ps = null;	
		 ResultSet rs = null;
		 try {
			 conn=  getConnect();
			 
			 String query = "SELECT quantity FROM shares WHERE ssn=? AND symbol=?";
			 ps = conn.prepareStatement(query);
			 ps.setString(1, ssn);
			 ps.setString(2, symbol);
			 
			 rs = ps.executeQuery();
			 if(rs.next()) {
				 int q=rs.getInt(1); //q는 현재 가지고 있는 주식의 수량
				 int newQuantity = q+quantity; //  q(50) + quantity(100)
				 
				 //UPDATE
				 String query1 = "UPDATE shares SET quantity=? WHERE ssn=? AND symbol=?";
				 ps = conn.prepareStatement(query1);
				 ps.setInt(1, newQuantity);
				 ps.setString(2, ssn);
				 ps.setString(3, symbol);
				 
				 System.out.println(ps.executeUpdate()+" row buyShares()....UPDATE OK");
			 }else { //주식이 없는 경우..
				 //INSERT
				 String query2 ="INSERT INTO shares (ssn, symbol, quantity) VALUES(?,?,?)";
				 ps = conn.prepareStatement(query2);
				 ps.setString(1, ssn);
				 ps.setString(2, symbol);
				 ps.setInt(3, quantity);
				 
				 System.out.println(ps.executeUpdate()+" row buyShares()....INSERT OK");
			 }
		 }finally {
			 closeAll(rs, ps, conn);
		 }
		
	}
/*
 * 누가 어떤 주식을 몇개 팔것인가에 대한 기능을 정의...
 * 현재 가지고 있는 주식의 수량을 먼저 알아야 한다....int q
 * 
 * 1) 100개를 가지고 있다 ---- 50  SELL --- update
 * 2) 100개를 가지고 있다 ---- 100 SELL --- delete
 * 3) 100개를 가지고 있다 ---- 200 SELL --- InvalidTransactionException 펑!! 
 * 4) 팔려는 주식이 없을때 				 --- RecordNotFoundException 펑!!
 */
	@Override
	public void sellShares(String ssn, String symbol, int quantity)
			throws SQLException, InvalidTransactionException {
		
		Connection conn = null;
		 PreparedStatement ps = null;	
		 ResultSet rs = null;
		 
		 try {
			 conn=  getConnect();
			 
			 String query ="SELECT quantity FROM shares WHERE ssn=? AND symbol=?";
			 ps = conn.prepareStatement(query);
			 ps.setString(1, ssn);
			 ps.setString(2, symbol);
			 
			 rs = ps.executeQuery();
			 
			 rs.next();//일단 커서를 한단계 아래로 내려서 엘러먼트를 가리키게 하고 수량을 받아올 준비를 한다.
			 
			 int q = rs.getInt(1); // 현재 가지고 있는 수량...100
			 int newQuantity = q-quantity; //팔고 남은 수량
			 
			 if(q==quantity) { //delete
				 String query1 = "DELETE FROM shares WHERE ssn=? AND symbol=?";
				 ps = conn.prepareStatement(query1);
				 ps.setString(1, ssn);
				 ps.setString(2, symbol);
				 
				 System.out.println(ps.executeUpdate()+" row SHARES DELETE....sellShares()1...");
			 }else if(q>quantity) { //update
				 String query2 = "UPDATE shares SET quantity=? WHERE ssn=? AND symbol=?";
				 ps = conn.prepareStatement(query2);
				 
				 ps.setInt(1, newQuantity);
				 ps.setString(2, ssn);
				 ps.setString(3, symbol);
				 
				 System.out.println(ps.executeUpdate()+" row SHARES UPDATE....sellShares()2...");
			 }else {  //펑
				 throw new InvalidTransactionException();
			 }		 			 
		 }finally {
			 closeAll(rs, ps, conn);
		 }
	}
}
----------------------------------------------------------------------

package broker.twotier.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Vector;

import broker.twotier.exception.DuplicateSSNException;
import broker.twotier.exception.InvalidTransactionException;
import broker.twotier.exception.RecordNotFoundException;
import broker.twotier.vo.CustomerRec;
import broker.twotier.vo.SharesRec;
import broker.twotier.vo.StockRec;

public interface DatabaseTemplate {
	Connection getConnect() throws SQLException;
	void closeAll(PreparedStatement ps, Connection conn)throws SQLException;
	void closeAll(ResultSet rs, PreparedStatement ps, Connection conn)throws SQLException;
	
	//비지니스 로직...CRUD
	void addCustomer(CustomerRec cust)throws SQLException,DuplicateSSNException;
	void deleteCustomer(String ssn)throws SQLException,RecordNotFoundException;
	void updateCustomer(CustomerRec cust)throws SQLException,RecordNotFoundException;
	
	Vector<SharesRec> getPortfolio(String ssn) throws SQLException,RecordNotFoundException;
	CustomerRec getCustomer(String ssn) throws SQLException,RecordNotFoundException;
	ArrayList<CustomerRec> getAllCustomers() throws SQLException,RecordNotFoundException;
	ArrayList<StockRec> getAllStocks() throws SQLException;
	
	//비지니스 로직...특화된 로직
	void buyShares(String ssn, String symbol, int quantity)throws SQLException;
	void sellShares(String ssn, String symbol, int quantity)throws SQLException,InvalidTransactionException,RecordNotFoundException;

}

---------------------------------------------------------------------

package broker.twotier.gui;
import java.awt.BorderLayout;
import java.awt.Button;
import java.awt.Color;
import java.awt.Frame;
import java.awt.GridLayout;
import java.awt.Label;
import java.awt.List;
import java.awt.Panel;
import java.awt.TextArea;
import java.awt.TextField;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.ItemEvent;
import java.awt.event.ItemListener;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.StringTokenizer;
import java.util.Vector;

import broker.twotier.dao.DatabaseSample;
import broker.twotier.exception.DuplicateSSNException;
import broker.twotier.exception.InvalidTransactionException;
import broker.twotier.exception.RecordNotFoundException;
import broker.twotier.vo.CustomerRec;
import broker.twotier.vo.SharesRec;
import broker.twotier.vo.StockRec;

//인터페이스 implements 한 상태로 클래스 선언하자
public class Broker implements ActionListener,ItemListener{
	private static int mode = 0;
	private static final int ADD_MODE = 1;
	private static final int UPDATE_MODE = 2;

	DatabaseSample		db;

	Frame 	frame =new Frame("ABC Stock");

	//*************컴포넌트 선언...생성 ************************
    //*******************************************************
	Panel 	pc =new Panel();
	Panel 	pe =new Panel();
	
	Panel 	   pec	=new Panel();
	Panel 	   pes	=new Panel();	
	
	
	Panel 	   pcn	=new Panel();
	Panel 	   pcc	=new Panel();
	Panel 	   pcw	=new Panel();
	Panel 	   pcs	=new Panel();
	

	Panel 		pcn1	=new Panel();
	Panel 		pcc1	=new Panel();
	Panel 		pcc2	=new Panel();
	
	Panel 		pcwn	=new Panel();
	Panel 		pcws	=new Panel();
	Panel 		pcwa	=new Panel();
	

	TextField nameTf	= new TextField(15);
	TextField ssnTf		= new TextField(15);
	TextField stockTf	= new TextField(15);
	TextField buyTf		= new TextField(15);
	TextField priceTf	= new TextField(15);
	TextField sellTf	= new TextField(15);

	TextArea addrTa	= new TextArea(3,15);

	List 	custList	= new List(15, false);
	List 	stockList	= new List(15, false);
	List 	portList	= new List(15, false);
	
	Button 	addB	= new Button("ADD");
	Button 	deleteB	= new Button("DELETE");
	Button 	updateB	= new Button("UPDATE");
	Button 	applyB	= new Button("apply");
	Button 	cancelB	= new Button("cancel");

	Button 	buyB	= new Button("Buy");
	Button 	sellB	= new Button("Sell");

	Button 	priceB	= new Button("Get Current Stock Price");

	// **************** 생성자 **************************************
	//*****************************************************************
	public Broker() throws Exception{
		
	   try	{	    
	    db =  new DatabaseSample("127.0.0.1");
		} catch(Exception cnfe) {
			System.out.println("Broker Constructor : " + cnfe);
	
		}
		createGUI();
		addListener();

		showCustList(db.getAllCustomers(),custList);
		showStockList(db.getAllStocks(),stockList);
	
		initButton(true);
		emptyText();
		textEditable(false);

	}//생성자 닫고...

	// **********배경색, 전경색 지정 **********************
	//**************************************************
	public void createGUI() {	
	
		pc.setBackground(new Color(196 ,196  ,255  ));
		pe.setBackground(new Color(196 ,196  ,255  ));
		pec.setBackground(new Color(196 ,196  ,255  ));
		pes.setBackground(new Color(196 ,196  ,255  ));
		pcn.setBackground(new Color(196 ,196  ,255  ));
		pcc.setBackground(new Color(196 ,196  ,255  ));
		pcw.setBackground(new Color(196 ,196  ,255  ));
		pcs.setBackground(new Color(196 ,196  ,255  ));
		pcn1.setBackground(new Color(196 ,196  ,255  ));
		pcc1.setBackground(new Color(196 ,196  ,255  ));
		pcc2.setBackground(new Color(196 ,196  ,255  ));
		pcwn.setBackground(new Color(196 ,196  ,255  ));
		pcws.setBackground(new Color(196 ,196  ,255  ));
		pcwa.setBackground(new Color(196 ,196  ,255  ));
		custList.setBackground(new Color(142 ,142  ,255));
		stockList.setBackground(new Color(48 ,0  ,96));
		portList.setBackground(new Color(142 ,142  ,255));
		sellTf.setBackground(new Color(196 ,196  ,255));
	
	    frame.add(pc,"Center");
		frame.add(pe,"East");
		// *******************  컴포넌트 부착  ************************************
		// **********************************************************************
		pe.setLayout(new BorderLayout());
			pe.add(new Label("Stock Information", Label.CENTER), "North");
			pe.add(pec, "Center");
			pe.add(pes, "South");

			pec.setLayout(new BorderLayout());
			pec.add(new Label("Available Stocks"), "North");
			pec.add(stockList, "Center");
			pec.add(priceB, "South");
			pes.setLayout(new GridLayout(2, 2));
			pes.add(new Label("  Stock"));	
			pes.add(stockTf);
			pes.add(new Label("  Current Price"));	
			pes.add(priceTf);

		pc.setLayout(new BorderLayout());
			pc.add(pcn, "North");
			pc.add(pcc, "Center");
			pc.add(pcw, "West");
			pc.add(pcs, "South");

			GridLayout grid = new GridLayout(2, 1);
			pcn.setLayout(grid);
			grid.setHgap(20);
			pcn.add(new Label("Customer Information", Label.CENTER));
			pcn.add(pcn1);
			pcn1.add(addB);
			pcn1.add(deleteB);
			pcn1.add(updateB);
			pcn1.add(applyB);
			pcn1.add(cancelB);

			pcc.setLayout(new GridLayout(1, 2));
			pcc.add(pcc1);
			pcc1.setLayout(new BorderLayout());
			pcc1.add(new Label("Stock Portfolio"), "North");
			pcc1.add(portList);

			pcc.add(pcc2);
			pcc2.setLayout(new BorderLayout());
			pcc2.add(new Label("All Customers"), "North");
			pcc2.add(custList);

			pcw.setLayout(new GridLayout(3, 1));
			pcw.add(pcwn);
			pcwn.add(new Label("Name"));
			pcwn.add(nameTf);
			pcw.add(pcws);
			pcws.add(new Label("SSN"));
			pcws.add(ssnTf);
			pcw.add(pcwa);
			pcwa.add(new Label("Address"));
			pcwa.add(addrTa);

			pcs.add(buyB);
			pcs.add(buyTf);
			pcs.add(sellTf);
			pcs.add(sellB);

	// ************* 버튼 초기화 *****************************************
	// ****************************************************************
		buyB.setEnabled(true);
		sellB.setEnabled(true);

		stockTf.setEditable	(false);
		priceTf.setEditable	(false);
		buyTf.setEditable  	(false);

	
		frame.setSize(700, 350);
		frame.setLocation(100, 100);
		frame.setVisible(true);
	}//createGUI() 닫고


	// ************** 리스너 부착 ****************************************
	//*****************************************************************

    public void addListener()
	{
		addB.addActionListener(this);
		deleteB.addActionListener(this);
        updateB.addActionListener(this);
        applyB.addActionListener(this);
		cancelB.addActionListener(this);
		buyB.addActionListener(this);
        sellB.addActionListener(this);
        priceB.addActionListener(this);

		custList.addItemListener(this);
		stockList.addItemListener(this);
        portList.addItemListener(this);               

            
		//***********프레임 창 닫는 로직. ***********************************
		//****************************************************************
   		frame.addWindowListener(
			new WindowAdapter()	{	
				public void windowClosing(WindowEvent we){	
					System.exit(0);
				}
			}
		);
	} //addListener() 닫고....

	/*
	버튼을 Group(add, delete, update vs apply, cancel)하여
	Enable되게 하는 메소드
	*/
    public void initButton(boolean b){
		addB.setEnabled(b);
        deleteB.setEnabled(b);
        updateB.setEnabled(b);
		applyB.setEnabled(!b);
		cancelB.setEnabled(!b);
	}
	// name, ssn, address  TextField의 편집상태를 바꾼다
	public void textEditable(boolean b)	{
		nameTf.setEditable	(b);
		ssnTf.setEditable	(b);
		addrTa.setEditable	(b);
	}

	//ssn, name, address의 TextFiled 값을 clear 시킨다.
	public void emptyText(){
    	nameTf.setText("");
    	ssnTf.setText("");
    	addrTa.setText("");                              
    }
	//==================================================================
	// Database의 method를 호출
	//==================================================================

		// showList(db.getAllCustomer() , custList )
		/**
		 * 1)customer List area에 있는 모든걸 지운다<BR>
		 * 2)CustomerRec[]에 있는 모든 CustomerRec 객체 내용을 List에 뿌려준다<BR>
		 */
		public void showCustList(ArrayList<CustomerRec> cust, List list){
			list.removeAll();
			for(CustomerRec c : cust) {
				String ssn = c.getSsn();
				String name = c.getName();
				String addr = c.getAddress();
				
				list.add(ssn+"  "+name+"  "+addr);				
			}			
	    }	
		
		/**
		*argument로 받은 CustomerRec[]을 stockList에 하나씩 뿌려준다.<BR>
		 * 1)stock List area에 있는 모든걸 지운다.<BR>
		 * 2)StockRec[]에 있는 모든 StockRec 객체 내용을 List에 뿌려준다.<BR>
		 */
		public void showStockList(ArrayList<StockRec> sr, List list){
			list.removeAll();
			list.setForeground(Color.YELLOW);
			for(StockRec s : sr) {
				String symbol = s.getSymbol();
				float price = s.getPrice();
				list.add(symbol+" "+price);
			}			
		}

		
		 /**
		<PRE>
		 * 1)인자값으로 입력된 Vector타입의 portfolio 정보를 폼 리스트중 Stock Portfolio에 뿌려준다.
		 </PRE>
		 */    
		public void showList(Vector<SharesRec> portfolio, List list){
			list.removeAll();
			
			for(SharesRec s : portfolio) {
				String symbol = s.getSymbol();
				int quantity = s.getQuantity();
				list.add(symbol +" "+quantity);
			}			
		}
		
		 /**
		<PRE>
		 * 1)customer List에서 선택된 항목중에서 ssn을 Token한다
		 * 2)잘라진 ssn으로 DB의 getCustomer()를 이용. table에서 ssn에 해당하는 나머지 정보를 가져온다
		 * 3)가져온 정보를 ssn,name,address TextField와 port LIst에 뿌린다. 
		 </PRE>
		 * @throws RecordNotFoundException 
		 */    
		public void showCustomer() throws RecordNotFoundException{
			String customer=custList.getSelectedItem();
			StringTokenizer st = new StringTokenizer(customer);
			String ssn = st.nextToken();
			System.out.println(ssn);
			try{
				CustomerRec cr=db.getCustomer(ssn);
				nameTf.setText(cr.getName());
				ssnTf.setText(cr.getSsn());
				addrTa.setText(cr.getAddress());
				Vector<SharesRec> v=cr.getPortfolio();
				if(v !=null){ //주식을 보유한 고객이라면...
					showList(v, portList);
				}else{ //주식을 보유하지 않은 고객이라면...
					portList.removeAll();
				}
			}catch(SQLException e){
				System.out.println("찾는 고객의 정보가 없어여...showCustomer()...");
			}
			
			
		} 
		
		 /**
		<PRE>
		 * 1)ssn, symbol, quantity 정보를 알아온다 --> ssnTf, buyTf, sellTf의 텍스트박스에 입력된 값
		 * 2)각각의 값들을 인자로 DB의 buyShares()를 이용. 
		 * 3)폼의 Stock Portfolio에 주식의 정보와 수량이 뿌려지게 한다.
		 </PRE>
		 */   
		public void buyStock(){
			String ssn = ssnTf.getText().trim();
			System.out.println(ssn+"...buyStock()...");
			String symbol = buyTf.getText().trim();
			int quantity = Integer.parseInt(sellTf.getText());
			try{
				db.buyShares(ssn, symbol, quantity);
				try {
					showCustomer();
				} catch (RecordNotFoundException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}//이거 안해주면 어떻게 되는 지 확인...
			}catch(SQLException e){
				e.printStackTrace();
			}				
			
		 } 
	  

		 /**
		<PRE>
		 * 1)ssn, symbol, quantity 정보를 알아온다 --> ssnTf, buyTf, sellTf의 텍스트박스에 입력된 값
		 * 2)각각의 값들을 인자로 DB의 sellShares()를 이용. 
		 * 3)폼의 Stock Portfolio에 주식의 정보와 수량이 뿌려지게 한다.
		 </PRE>
		 */   
		public void sellStock() throws RecordNotFoundException{
			String ssn = ssnTf.getText().trim();
			System.out.println(ssn+"...sellStock()...");
			String symbol = buyTf.getText().trim();
			int quantity = Integer.parseInt(sellTf.getText());
			try{
				db.sellShares(ssn, symbol, quantity);
				
			}catch(SQLException e){
				e.printStackTrace();
			}catch(InvalidTransactionException e2){
				System.out.println("팔려는 주식이 넘 많아여...sellStock()");
			}
			try {
				showCustomer();
			} catch (RecordNotFoundException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}//이거 안해주면 어떻게 되는 지 확인...
		}
		 
		 /**
		<PRE>
		 * 1)apply button에 의해 호출되는 메소드(Add button과 관련있다.)
		 * 2)nameTf,ssnTf,addrTa 에 입력된 값을 받아와 CustomerRec 객체를 생성
		 * 3)DB의 addCustomer()를 호출하고
		 * 4)최종적으로 List에 추가된 고객을 포함한 모든 고객의 리스트를 뿌린다.
		 </PRE>
		 * @throws RecordNotFoundException 
		 */
		public void addCustomer() throws RecordNotFoundException{
			
			String name = nameTf.getText().trim();
			String ssn = ssnTf.getText().trim();
			String address = addrTa.getText().trim();
			CustomerRec cr = new CustomerRec(ssn, name, address);
			
			try{
				db.addCustomer(cr);
				showCustList(db.getAllCustomers(), custList);
			}catch(DuplicateSSNException e){
				System.out.println("중복되는 고객 있어여...addCustomer()");
			}catch(SQLException e){
				e.printStackTrace();
			}
			
	    } 
		 /**
		<PRE>
		 * 1)apply button에 의해 호출되는 메소드(Add button과 관련있다.)
		 * 2)nameTf,ssnTf,addrTa 에 입력된 값을 받아와 CustomerRec 객체를 생성
		 * 3)DB의 updateCustomer()를 호출하고
		 * 4)최종적으로 List에 수정된 고객을 포함한 모든 고객의 리스트를 뿌린다.
		 </PRE>
		 */ 		  
		public void updateCustomer(){
			
			String name = nameTf.getText().trim();//trim은 양쪽 공백을 제거해주는 기능
			String ssn = ssnTf.getText().trim();
			String address = addrTa.getText().trim();
			CustomerRec cr = new CustomerRec(ssn, name, address);
			
			try{
				db.updateCustomer(cr);
				showCustList(db.getAllCustomers(), custList);
			}catch(RecordNotFoundException e){
				System.out.println("수정할 대상의 고객이 없습니다....");
			}catch(SQLException e){
				e.printStackTrace();
			}
			
	    } 
		
		/**
		//delete button에 의해 호출된다.
		 * 1)database의 deleteCustomer(ssn)call<BR>
		 * 2)showList(CustomerRec[],List) 호출<BR>
		 * 
		 */
		public void deleteCustomer(){
			String ssn = ssnTf.getText().trim();
			try{
				db.deleteCustomer(ssn);
				ArrayList<CustomerRec> list = db.getAllCustomers();
				showCustList(list, custList);
			}catch(Exception e){
				System.out.println("삭제하려는 고객이 없습니다. Broker.deleteCustomer() "+e);
			}
	    } 



		
		/**
		<PRE>
		*stockList에 선택된 내용을
		*buyTf, sockTf, priceTf TextField에 뿌려준다.
		 * 1)stock List 중에서 선택된 항목을 Token한다.
		 * 2)symbol,price를 해당 textfield에 setting 한다.
		 </PRE>
		 */
		public void showStock(){
			String stock = stockList.getSelectedItem();
			StringTokenizer st = new StringTokenizer(stock);
			String symbol = st.nextToken().trim();
			String price = st.nextToken().trim();
			
			buyTf.setText(symbol);
			stockTf.setText(symbol);
			priceTf.setText(price);
		}

		  /**
		*portList에 선택된 내용을 buy, sell TextField에 뿌려준다.
		 * 1)port List에서 선택된 항목을 Token한다.<BR>
		 * 2)symbol,quantity를 해당 textfield에 setting한다<BR>
		 */
		public void showPortfolio(){
			String portfolio = portList.getSelectedItem();
			StringTokenizer st = new StringTokenizer(portfolio);
			String symbol = st.nextToken();
			String quantity = st.nextToken();
			
			buyTf.setText(symbol);
			sellTf.setText(quantity);
	   	}


	//============================================================
	// Event Handling 처리
	//=============================================================

		 /**
		<PRE>
		 * List내에서 다른 아이템을 선택하면 call
		 * 1)이 메소드 호출시 언제나 2개의 textfield(buy,sell)의 상태를 null로 만들것
		 * 2)이 메소드를 호출시킨 event source가
		 *   customer List일 경우 : showCustomer() method call
		 *   portfolio List일 경우 : showPortfolio() method call
		 *   stock List 일 경우 : showStock() method call
		</PRE>
		 */
		public void itemStateChanged(ItemEvent ie) {
			buyTf.setText("");
			sellTf.setText("");
			List list = (List)ie.getSource();
			if(list.equals(custList)){
				try {
					showCustomer();
				} catch (RecordNotFoundException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}else if(list.equals(stockList)){
				showStock();
			}else{
				showPortfolio();
			}		
			
		}
		//Button들이 eventHandling
		public void actionPerformed(ActionEvent ae){
			String buttonLabel = ae.getActionCommand();
			if(buttonLabel.equals("ADD")){
				initButton(false);
				textEditable(true);
				emptyText(); 
				nameTf.requestFocus();
				mode = ADD_MODE;
				System.out.println(mode);
			}else if(buttonLabel.equals("DELETE")){
				deleteCustomer();
				emptyText();
			}else if(buttonLabel.equals("UPDATE")){
				initButton(false);
				textEditable(true);
				nameTf.requestFocus();
				mode = UPDATE_MODE;
				System.out.println(mode);
			}else if (buttonLabel.equals("apply"))	{
			
				switch(mode){
					case ADD_MODE:
					try {
						addCustomer();
					} catch (RecordNotFoundException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
						emptyText();
						textEditable(false);
						initButton(true);
						break;
					case UPDATE_MODE:
						updateCustomer();
						textEditable(false);
						initButton(true);
						break;
				}
			}else if(buttonLabel.equals("cancel")){
				initButton(true);
				emptyText();
				textEditable(false);
			}else if(buttonLabel.equals("Buy")){
				buyStock();
				sellTf.setText("");
			}else if (buttonLabel.equals("Sell")){
				try {
					sellStock();
				} catch (RecordNotFoundException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				sellTf.setText("");
				System.out.println(0);
			}
		}
	public static void main(String args[])throws Exception {
		Broker broker = new Broker();		
	}
}

--------------------------------------------------------------------

package broker.twotier.test;

import java.util.ArrayList;
import java.util.Vector;

import broker.twotier.dao.Database;
import broker.twotier.vo.CustomerRec;
import broker.twotier.vo.SharesRec;
import broker.twotier.vo.StockRec;

public class DatabaseTest {

	public static void main(String[] args) throws Exception {
		
		Database db = new Database("127.0.0.1");
		
		//db.addCustomer(new CustomerRec("888-888","sujin","방배동"));
		
		//db.deleteCustomer("888-888");
		
		//db.updateCustomer(new CustomerRec("111-118", "Kim", "성수동"));
		
		//Vector<SharesRec> list = db.getPortfolio("111-115");
		
		//db.getCustomer("111-118");
		
//		ArrayList<CustomerRec> list=db.getAllCustomers();
//		for(CustomerRec cr : list) {
//			System.out.println(cr);
		
//		ArrayList<StockRec> list = db.getAllStocks();
//		for(StockRec sr : list ) {
//			System.out.println(sr);
//		}
		
	 //db.buyShares("111-115", "JDK", 60);
		
		//db.sellShares("111-118", "JDK", 10);
	}

}
-------------------------------------------------------



