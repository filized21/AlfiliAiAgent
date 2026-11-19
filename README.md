from pathlib import Path
import logging
import sys

PROJECT_DIR = Path(__file__).resolve().parent
LOG_DIR = PROJECT_DIR / "logs"
MEMORY_DIR = PROJECT_DIR / "memory"
OUTPUT_DIR = PROJECT_DIR / "outputs"

for folder in [LOG_DIR, MEMORY_DIR, OUTPUT_DIR]:
    folder.mkdir(parents=True, exist_ok=True)

sys.path.insert(0, str(PROJECT_DIR / "core"))
sys.path.insert(0, str(PROJECT_DIR / "tools"))

from memory import add_task, list_tasks, delete_task
from system_tools import run_system_check
from file_tools import list_directory, create_folder, read_text_file
from search_tools import search_files

logging.basicConfig(
    filename=LOG_DIR / "agent.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    encoding="utf-8"
)


def show_status():
    print("\n===== ALFILI AI AGENT =====")
    print("Version: 0.6")
    print("Status: Running")
    print("Persistent memory: Ready")
    print("System tools: Ready")
    print("File tools: Ready")
    print("Search tools: Ready")
    print("Project:", PROJECT_DIR)


def add_memory_task():
    description = input("\nاكتب المهمة التي تريد حفظها: ").strip()

    if not description:
        print("لم تكتب أي مهمة.")
        return

    try:
        task = add_task(description)
        print("\nتم حفظ المهمة:", task["description"])
        logging.info("Memory task added")
    except (ValueError, OSError) as error:
        print("تعذّر حفظ المهمة:", error)


def show_memory_tasks():
    try:
        tasks = list_tasks()
        print("\n===== SAVED MEMORY =====")

        if not tasks:
            print("لا توجد مهام محفوظة.")
            return

        for number, task in enumerate(tasks, start=1):
            print(f"{number} - {task['description']}")
            print(f"    Date: {task['created_at']}")

    except (ValueError, OSError) as error:
        print("تعذّرت قراءة الذاكرة:", error)


def remove_memory_task():
    show_memory_tasks()

    try:
        tasks = list_tasks()

        if not tasks:
            return

        number = int(input("\nأدخل رقم المهمة التي تريد حذفها: "))
        deleted = delete_task(number)

        if deleted:
            print("\nتم حذف المهمة:", deleted["description"])
            logging.info("Memory task deleted")
        else:
            print("\nرقم المهمة غير صحيح.")

    except ValueError:
        print("\nأدخل رقماً صحيحاً.")
    except (OSError, KeyError) as error:
        print("تعذّر حذف المهمة:", error)


def show_folders():
    print("\nProject folders:")
    for folder in sorted(PROJECT_DIR.iterdir()):
        if folder.is_dir():
            print("-", folder.name)


def file_manager_menu():
    while True:
        print("\n===== FILE MANAGER =====")
        print("1 - List files and folders")
        print("2 - Create a folder")
        print("3 - Read a text file")
        print("4 - Search for files")
        print("0 - Back to main menu")

        choice = input("\nChoose an option: ").strip()

        if choice == "1":
            folder_path = input(
                "Folder path (blank = project root): "
            ).strip() or "."

            try:
                items = list_directory(folder_path)
                print("\n===== DIRECTORY CONTENTS =====")

                if not items:
                    print("This folder is empty.")
                else:
                    for item in items:
                        print(f"{item['type']}: {item['name']}")

            except (OSError, ValueError) as error:
                print("Could not list folder:", error)

        elif choice == "2":
            folder_path = input("New folder path: ").strip()

            try:
                name = create_folder(folder_path)
                print("Folder created:", name)
                logging.info("Project folder created")

            except (OSError, ValueError) as error:
                print("Could not create folder:", error)

        elif choice == "3":
            file_path = input("Text file path: ").strip()

            try:
                content = read_text_file(file_path)
                print("\n===== FILE CONTENT =====")
                print(content)

            except (OSError, ValueError, UnicodeError) as error:
                print("Could not read file:", error)

        elif choice == "4":
            query = input("File name to search for: ").strip()
            folder_path = input(
                "Search folder (blank = project root): "
            ).strip() or "."

            try:
                results = search_files(query, folder_path)

                print("\n===== SEARCH RESULTS =====")

                if results:
                    for result in results:
                        print("-", result)
                    print("Files found:", len(results))
                else:
                    print("No matching files found.")

            except (OSError, ValueError) as error:
                print("Search failed:", error)

        elif choice == "0":
            break

        else:
            print("Invalid option. Try again.")


def show_menu():
    print("\n===== ALFILI AI AGENT =====")
    print("1 - Check system status")
    print("2 - List project folders")
    print("3 - Save a task in memory")
    print("4 - View saved tasks")
    print("5 - Delete a task from memory")
    print("6 - Run system tools")
    print("7 - File manager")
    print("0 - Exit")


def main():
    print("Welcome to Alfili AI Agent")
    logging.info("Agent started")

    while True:
        show_menu()
        choice = input("\nChoose an option: ").strip()

        if choice == "1":
            show_status()

        elif choice == "2":
            show_folders()

        elif choice == "3":
            add_memory_task()

        elif choice == "4":
            show_memory_tasks()

        elif choice == "5":
            remove_memory_task()

        elif choice == "6":
            try:
                run_system_check()
                logging.info("System check completed")
            except Exception as error:
                print("System check failed:", error)
                logging.exception("System check failed")

        elif choice == "7":
            file_manager_menu()

        elif choice == "0":
            logging.info("Agent stopped")
            print("\nAlfili AI Agent stopped.")
            break

        else:
            print("\nInvalid option. Try again.")


if __name__ == "__main__":
    main()
