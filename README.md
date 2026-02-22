from datetime import datetime, timedelta

class Book:
    def __init__(self, title, author, quantity=1):
        self.title = title
        self.author = author
        self.quantity = quantity
        self.available = quantity
    
    def borrow(self):
        """Borrow a copy of this book if available."""
        if self.available > 0:
            self.available -= 1
            return True
        return False

    def return_book(self):
        """Return a copy of this book."""
        if self.available < self.quantity:
            self.available += 1
            return True
        return False

    def __str__(self):
        return f'{self.title} by {self.author} (Available: {self.available}/{self.quantity})'
    
class Librarian:
    def __init__(self, staff_name, gender):
        self.staff_name = staff_name
        self.gender = gender

    def __str__(self):
        return f'{self.staff_name} ({self.gender})'
    
class Member:
    def __init__(self, name, level):
        self.name = name
        self.level = level
        self.borrowed_books = {}

    def __str__(self):
        return f'{self.name} - Level: {self.level}'

    def borrow_book(self, book, days=7):
        """Borrow a book and track the due date."""
        if not book.borrow():
            return f'ERROR: {book.title} is not available'
        
        borrow_date = datetime.now()
        return_date = borrow_date + timedelta(days=days)
        
        self.borrowed_books[book.title] = {
            "book": book,
            "borrow_date": borrow_date,
            "return_date": return_date,
            "returned": False
        }
        return f'{book.title} borrowed successfully (Due: {return_date.strftime("%Y-%m-%d")})'

    def return_book(self, title):
        """Return a borrowed book."""
        if title not in self.borrowed_books:
            return f'ERROR: {title} not found in your borrowed books'
        
        if self.borrowed_books[title]['returned']:
            return f'ERROR: {title} was already returned'
        
        self.borrowed_books[title]['book'].return_book()
        self.borrowed_books[title]['returned'] = True
        return f'{title} returned successfully'

    def track_overdue(self):
        """Calculate late fees for overdue books."""
        today = datetime.now()
        fee = 0
        overdue_books = []
        
        for title, record in self.borrowed_books.items():
            if not record['returned'] and today > record['return_date']:
                late_days = (today - record['return_date']).days
                fee += late_days * 4
                overdue_books.append(f'{title} ({late_days} days late)')
        
        if overdue_books:
            return f'Overdue Books: {", ".join(overdue_books)} | Late Fee: ${fee}'
        return f'No overdue books. Late Fee: $0'
    
class Library:
    def __init__(self):
        self.books = []
        self.members = []
        self.librarians = []

    def add_book(self, book):
        """Add a book to the library."""
        if not isinstance(book, Book):
            return 'ERROR: Invalid book object'
        self.books.append(book)
        return f'✓ Book added: {book.title}'
    
    def add_member(self, member):
        """Add a member to the library."""
        if not isinstance(member, Member):
            return 'ERROR: Invalid member object'
        self.members.append(member)
        return f'✓ Member added: {member.name}'
    
    def add_librarian(self, librarian):
        """Add a librarian to the staff."""
        if not isinstance(librarian, Librarian):
            return 'ERROR: Invalid librarian object'
        self.librarians.append(librarian)
        return f'✓ Librarian added: {librarian.staff_name}'
    
    def search_by_title(self, book_name):
        """Search for books by title."""
        results = [book for book in self.books if book.title.lower() == book_name.lower()]
        if results:
            return results
        return None
    
    def search_by_author(self, author_name):
        """Search for books by author."""
        results = [book for book in self.books if book.author.lower() == author_name.lower()]
        if results:
            return results
        return None

    def search_for_members(self, member_name):
        """Search for a member by name."""
        results = [member for member in self.members if member.name.lower() == member_name.lower()]
        if results:
            return results
        return None

    def display_books(self):
        """Display all books in the library."""
        if not self.books:
            return 'No books in the library'
        
        output = '\n=== LIBRARY BOOKS ===\n'
        for i, book in enumerate(self.books, 1):
            output += f'{i}. {book}\n'
        return output

    def display_members(self):
        """Display all members."""
        if not self.members:
            return 'No members registered'
        
        output = '\n=== LIBRARY MEMBERS ===\n'
        for i, member in enumerate(self.members, 1):
            output += f'{i}. {member}\n'
        return output

    def display_librarians(self):
        """Display all librarians."""
        if not self.librarians:
            return 'No librarians on staff'
        
        output = '\n=== LIBRARY STAFF ===\n'
        for i, lib in enumerate(self.librarians, 1):
            output += f'{i}. {lib}\n'
        return output


def main_menu():
    """Main interactive menu for the library system."""
    library = Library()
    
    while True:
        print('\n=== LIBRARY MANAGEMENT SYSTEM ===')
        print('1. Add Book')
        print('2. Add Member')
        print('3. Add Librarian')
        print('4. Search Books by Title')
        print('5. Search Books by Author')
        print('6. Search Member')
        print('7. Display All Books')
        print('8. Display All Members')
        print('9. Display All Librarians')
        print('10. Member Borrow Book')
        print('11. Member Return Book')
        print('12. Check Overdue Fees')
        print('0. Exit')
        
        choice = input('\nEnter your choice: ').strip()
        
        if choice == '1':
            title = input('Book title: ').strip()
            author = input('Author: ').strip()
            qty = input('Quantity (default 1): ').strip()
            qty = int(qty) if qty.isdigit() else 1
            print(library.add_book(Book(title, author, qty)))
        
        elif choice == '2':
            name = input('Member name: ').strip()
            level = input('Member level: ').strip()
            print(library.add_member(Member(name, level)))
        
        elif choice == '3':
            name = input('Staff name: ').strip()
            gender = input('Gender: ').strip()
            print(library.add_librarian(Librarian(name, gender)))
        
        elif choice == '4':
            title = input('Search book title: ').strip()
            results = library.search_by_title(title)
            if results:
                for book in results:
                    print(f'Found: {book}')
            else:
                print('No results found')
        
        elif choice == '5':
            author = input('Search author: ').strip()
            results = library.search_by_author(author)
            if results:
                for book in results:
                    print(f'Found: {book}')
            else:
                print('No results found')
        
        elif choice == '6':
            name = input('Search member name: ').strip()
            results = library.search_for_members(name)
            if results:
                for member in results:
                    print(f'Found: {member}')
            else:
                print('No results found')
        
        elif choice == '7':
            print(library.display_books())
        
        elif choice == '8':
            print(library.display_members())
        
        elif choice == '9':
            print(library.display_librarians())
        
        elif choice == '10':
            member_name = input('Member name: ').strip()
            members = library.search_for_members(member_name)
            if members:
                member = members[0]
                book_title = input('Book title to borrow: ').strip()
                books = library.search_by_title(book_title)
                if books:
                    print(member.borrow_book(books[0]))
                else:
                    print('Book not found')
            else:
                print('Member not found')
        
        elif choice == '11':
            member_name = input('Member name: ').strip()
            members = library.search_for_members(member_name)
            if members:
                member = members[0]
                book_title = input('Book title to return: ').strip()
                print(member.return_book(book_title))
            else:
                print('Member not found')
        
        elif choice == '12':
            member_name = input('Member name: ').strip()
            members = library.search_for_members(member_name)
            if members:
                print(members[0].track_overdue())
            else:
                print('Member not found')
        
        elif choice == '0':
            print('Thank you for using Library Management System!')
            break
        
        else:
            print('Invalid choice. Please try again.')


if __name__ == '__main__':
    main_menu()
