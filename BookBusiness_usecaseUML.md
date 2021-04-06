# TIL_jdbc


package config;

public interface ServerInfo {
	String DRIVER="com.mysql.cj.jdbc.Driver";//public static final 무조건 생략
	String URL = "jdbc:mysql://127.0.0.1:3306/scott?serverTimezone=UTC&useUnicode=yes&characterEncoding=UTF-8";
	String USER= "root";
	String PASSWORD = "1234";
}

---------------------------------------------------
package com.encore.vo;
// db table에 대한 정보를 담고 있는 클래스..
//Book table의 하나의 row를 인스턴스화 시킨 클래스
//vo, domain, dto라 부른다.
public class Book {

	private String isbn;
	private String title;
	private String writer;
	private String publisher;
	private int price;
	
	public Book() {
		super();
		// TODO Auto-generated constructor stub
	}
	
	public Book(String isbn, String title, String writer, String publisher, int price) {
		super();
		this.isbn = isbn;
		this.title = title;
		this.writer = writer;
		this.publisher = publisher;
		this.price = price;
	}

	public String getIsbn() {
		return isbn;
	}

	public void setIsbn(String isbn) {
		this.isbn = isbn;
	}

	public String getTitle() {
		return title;
	}

	public void setTitle(String title) {
		this.title = title;
	}

	public String getWriter() {
		return writer;
	}

	public void setWriter(String writer) {
		this.writer = writer;
	}

	public String getPublisher() {
		return publisher;
	}

	public void setPublisher(String publisher) {
		this.publisher = publisher;
	}

	public int getPrice() {
		return price;
	}

	public void setPrice(int price) {
		this.price = price;
	}

	@Override
	public String toString() {
		return "Book [isbn=" + isbn + ", title=" + title + ", writer=" + writer + ", publisher=" + publisher
				+ ", price=" + price + "]";
	}
	
	
	
}
---------------------------------------
package com.encore.exception;

public class BookNotFoundException extends Exception {

	public BookNotFoundException() {
		this("책을 찾을 수 없습니다!");
	}

	public BookNotFoundException(String message) {
		super(message);
	}
	
}
-----------------------------------------

package com.encore.exception;

public class DuplicateISBNException extends Exception {

	public DuplicateISBNException() {
		this("ISBN이 중복되었습니다!");
	}

	public DuplicateISBNException(String message) {
		super(message);
	}
}
------------------------------------------

package com.encore.exception;

public class InvalidInputException extends Exception {
	
	public InvalidInputException() {
		this("유효하지 않은 결과입니다!");
	}

	public InvalidInputException(String message) {
		super(message);
	}
}

----------------------------------------------

package com.encore.dao;

import java.sql.SQLException;
import java.util.ArrayList;

import com.encore.exception.BookNotFoundException;
import com.encore.exception.DuplicateISBNException;
import com.encore.exception.InvalidInputException;
import com.encore.vo.Book;

public interface BookDAO {

	void registerBook(Book vo) throws SQLException, DuplicateISBNException;
	void deleteBook(String isbn) throws SQLException, BookNotFoundException;
	Book findByBook(String isbn, String title) throws SQLException;
	ArrayList<Book> findByWriter(String writer) throws SQLException;
	ArrayList<Book> findByPublisher(String publisher) throws SQLException;
	
	//가격에 대한 검색..
	ArrayList<Book> findByPrice(int min, int max) throws SQLException, InvalidInputException;
	void discountBook(int per, String oublisher) throws SQLException;//출판사별 할인율 
	ArrayList<Book> printAllInfo() throws SQLException;
    
	
	
	
}
---------------------------------------------

package com.encore.test;


import java.sql.SQLException;
import java.util.ArrayList;

import com.encore.dao.impl.BookDAOImpl;
import com.encore.vo.Book;



public class BookDAOTest {

	public static void main(String[] args) throws ClassNotFoundException,Exception, SQLException{
	
		//BookDAOImpl.getInstance().registerBook(new Book("7G7","미움받을용기","데이비드","동아",15000));
		//BookDAOImpl.getInstance().deleteBook("7G7");
		
		//ArrayList<Book> list = BookDAOImpl.getInstance().findByPublisher("동아");	
		//for(Book book : list) 
		//	System.out.println(book);
		
//		ArrayList<Book> list = BookDAOImpl.getInstance().findByPrice(10000, 30000);
//		for(Book book : list) 
//		System.out.println(book);
		
		//BookDAOImpl.getInstance().discountBook(10, "도우");
		
//		ArrayList<Book> list= BookDAOImpl.getInstance().printAllInfo();
//		for(Book book : list) 
//			System.out.println(book);
		
	//	System.out.println(BookDAOImpl.getInstance().findByBook("2B2", "몰입"));
	}
}

