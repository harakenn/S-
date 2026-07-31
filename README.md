# S-
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <sstream>
#include <regex>
#include <iomanip>
#include <ctime>
#include <fstream>
#include <sys/stat.h>

#ifdef _WIN32
#include <windows.h>
#endif

struct Schedule {
    std::string subject;   // 予定
    std::string startDate; // YYYY/MM/DD
    std::string endDate;   // YYYY/MM/DD
};

bool compareDesc(const Schedule& a, const Schedule& b) {
    return a.startDate > b.startDate;
}

// YYYY-MM-DD → YYYY/MM/DD
std::string formatDate(const std::string& s) {
    if (s.size() != 10) return s;
    return s.substr(0, 4) + "/" + s.substr(5, 2) + "/" + s.substr(8, 2);
}

int main() {
#ifdef _WIN32
    SetConsoleOutputCP(65001);
    SetConsoleCP(65001);
#endif

    std::vector<Schedule> schedules;
    std::string allInput, line;

    // 英語表示にすることでコンソールの文字化けを完全に回避
    std::cout << "========================================\n";
    std::cout << " Schedule to CSV Converter\n";
    std::cout << "========================================\n";
    std::cout << "[Example Format]\n";
    std::cout << "  2026-08-01 Meeting\n";
    std::cout << "  2026-08-03~2026-08-05 Summer Vacation\n";
    std::cout << "----------------------------------------\n";
    std::cout << "Paste the email body (Press Enter twice to finish):\n\n";

    while (true) {
        std::getline(std::cin, line);
        if (line.empty()) break;
        allInput += line + "\n";
    }

    std::regex re_multi(R"(^(\d{4}-\d{2}-\d{2})~(\d{4}-\d{2}-\d{2})\s+(.+)$)");
    std::regex re_single(R"(^(\d{4}-\d{2}-\d{2})\s+(.+)$)");
    std::smatch match;

    std::istringstream iss(allInput);
    while (std::getline(iss, line)) {
        if (std::regex_match(line, match, re_multi)) {
            std::string start = match[1];
            std::string end = match[2];
            std::string event = match[3];
            schedules.push_back({ event, formatDate(start), formatDate(end) });
        }
        else if (std::regex_match(line, match, re_single)) {
            std::string date = match[1];
            std::string event = match[2];
            schedules.push_back({ event, formatDate(date), formatDate(date) });
        }
    }

    std::sort(schedules.begin(), schedules.end(), compareDesc);

    // --- CSV出力 ---
    const std::string filename = "schedule.csv";
    std::ofstream ofs(filename, std::ios::binary);
    if (ofs) {
        // UTF-8 BOM
        const char bom[] = "\xEF\xBB\xBF";
        ofs.write(bom, 3);

        // ヘッダー
        std::string header = "Subject,Start Date,Start Time,End Date,End Time,All Day Event,Description\r\n";
        ofs.write(header.c_str(), header.size());

        // データ
        for (const auto& schedule : schedules) {
            std::string csvline = schedule.subject + "," +
                schedule.startDate + "," +
                "," +
                schedule.endDate + "," +
                "," +
                "TRUE,\r\n";
            ofs.write(csvline.c_str(), csvline.size());
        }
        ofs.close();
        std::cout << "\nSuccessfully exported to " << filename << " (UTF-8 with BOM).\n";
    }
    else {
        std::cerr << "Failed to create CSV file.\n";
    }

    return 0;
}
