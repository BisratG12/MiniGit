# MiniGit
```
#include <iostream>
#include <filesystem>
#include <unordered_map>
#include <vector>
namespace fs = std::filesystem;

// Simple hash using SHA-1 would require a library; here's a placeholder:
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
        std::cout << "Repo initialized at: " << repoPath << "\n";
    }

    void initRepo() {
        fs::create_directories(objectsPath);
        std::ofstream(headPath, std::ios::app);
        std::ofstream(indexPath, std::ios::trunc);
void add(const std::string& file) {
        std::ifstream fin(file);
        if (!fin) { std::cerr << "File not found\n"; return; }
        std::stringstream ss; ss << fin.rdbuf();
        std::string content = ss.str();
        std::string hash = simpleHash(content);
        fs::path obj = objectsPath / hash;
        std::ofstream(obj) << content;

        std::ofstream idx(indexPath, std::ios::app);
        idx << file << " " << hash << "\n";
        std::cout << "Added " << file << "\n";
    }
```
