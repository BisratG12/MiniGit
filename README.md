# MiniGit
```
#include <iostream>
#include <string>
#include <filesystem>
#include <fstream>
#include <unordered_map>
#include <vector>
#include <chrono>
#include <sstream>
#include <iomanip>
#include <map>
#include <set>
namespace fs = std::filesystem;
using namespace std;

// simple hash is usefull to create a unique key based on the content of the file so each versions has 
// there own uniqe key and it is stored inside the obejct folder  
std::string simpleHash(const std::string& data) {
    std::hash<std::string> h;
    return std::to_string(h(data));
}
class MiniGit {
    fs::path repoPath, objectsPath, headPath, indexPath;

public:
    MiniGit(const std::string& base = ".") {
        repoPath   = fs::path(base) / ".minigit";
        objectsPath = repoPath / "objects";
        headPath    = repoPath / "HEAD";
        indexPath   = repoPath / "index";
        initRepo();
        std::cout << "Repo is initialized at: " << repoPath << endl;
    }

    void initRepo() {
        fs::create_directories(objectsPath); // to create a folder that holds the versions
        std::ofstream(headPath, std::ios::app);
        std::ofstream(indexPath, std::ios::trunc);
    }

    void add(const std::string& file) {//make a version ready to be commited 
        std::ifstream fin(file);
        if (!fin) { std::cerr << "File not found\n"; return; }// if file is empty return file not found 
        std::stringstream ss; // it creates string container
        ss << fin.rdbuf();// it takes everything from the file and copy it in ss container 
        std::string content = ss.str();// it copies it to the content container why use two containers 
        // because ss << fin.rdbuf(); is efficient and it dump it once otherwise we use while loop 
        std::string hash = simpleHash(content);
        fs::path obj = objectsPath / hash;// it creates the address of this version 
        std::ofstream(obj) << content;// it creates the file and store the content it with the label of hash

        std::ofstream idx(indexPath, std::ios::app);// it adds a line at the end
        idx << file << " " << hash << "\n";// the line is that is added it has file name and hash  which make it ready for commit 
        std::cout << "Added " << file << "\n";
    }

    void commit(const std::string& msg) {// msg is the message the wrriten by the user
        std::ifstream reader(indexPath);//open file for reading 
        std::vector<std::string> lines;
        std::string line;
        while (std::getline(reader, line)){//read from the file and push it in to the vector
             lines.push_back(line);}

        std::string parent;// to store the previous commit hash
        std::ifstream headIn(headPath);
        std::getline(headIn, parent);

        auto now = std::chrono::system_clock::to_time_t(std::chrono::system_clock::now());// to store the current time 
        std::ostringstream commitContent;
        commitContent << std::put_time(std::localtime(&now), "%F %T") << "|" << msg << "\n";// Add formatted current time
        commitContent << parent << "\n";
        for (auto& l : lines) {//it writes each line in the lines 
            commitContent << l << "\n";
        }

        std::string data = commitContent.str();
        std::string hash = simpleHash(data);//create a unique hash id for this commit
        std::ofstream(objectsPath / hash) << data;// save the commit data in file named by hash

        std::ofstream(headPath, std::ios::trunc) << hash;//it opens head and erases everything inside then writes the new commits hash 
        std::ofstream(indexPath, std::ios::trunc);// it erases indexfile to make it ready for the next 
        std::cout << "Committed: " << hash << "\n";
    }

    void log() {
        std::string cur;
        std::ifstream headIn(headPath);// Open the HEAD file it tells the latest hash
        std::getline(headIn, cur);// read the commit hash

        while (!cur.empty()) {// go backward through commit history by following the parent links
            fs::path commitPath = objectsPath / cur;
            std::ifstream in(commitPath);
            if (!in) break;

            std::string metaLine, parent;
            std::getline(in, metaLine);  //date and message
            std::getline(in, parent);   // parent commit hash

            std::cout << "Commit " << cur << " | " << metaLine << "\n";

            cur = parent;
        }
    }
```
