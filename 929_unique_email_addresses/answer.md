問題：https://leetcode.com/problems/unique-email-addresses/description/

## step1

問題は
　1.emailの形をした（途中に'@'が入り、英小文字と'.'と'+'で構成された）文字列が入った配列emailsを渡される。
　2.それらの内、'@'より前のlocal部と後のdomain部に分けて、local部では'.'はあってもなくても同じ、'+'は+以降の文字をlocal部内でないのと同じに扱うという規則で何種類のemailがあるかを返り値として渡す。
というもの。

手でやる場合、local部で'.'と'+'から'@'の一つ手前の文字までを消してから比較するという考えから、それをそのまま実行するプログラムを書いた。
具体的には、emailsの各要素について、'+'と'@'の出現にフラッグでの判定を用い、
　1.'+'と'@'がまだどちらも出現していない場合(local)
　2.'+'のみ先に出現した場合(local)
　3.'@'のみ出現した場合(domain)
　4.'+'と'@'がどちらも出現した場合(domain)
で場合分けした。3と4の場合はdomain部なので何もせずに文字列を書き写せばよい。
個別のケースとして'.'が'@'の出現前に現れた場合も'.'を書き写してはいけないが、これは2の場合と処理を統合した。

```cpp
class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        unordered_set<string> email_set;
        for (string email : emails) {
            string email_key;
            bool at = false;
            bool plus = false;
            for (int i = 0; email[i] != '\0'; i++) {
                if (email[i] == '@') {
                    at = true;
                }
                if ((at == false && email[i] == '.') ||
                    (at == false && plus == true)) {
                    continue;
                }
                if (at == false && email[i] == '+') {
                    plus = true;
                    continue;
                }
                email_key.push_back(email[i]);
            }
            email_set.insert(email_key);
        }
        return email_set.size();
    }
};
```

計算量は空間がO(NW)で、時間がO(NW)になると思う。
空間計算量ではWが最大文字数だとして、最悪の場合は全入力データをhashsetで保持するから。
また、二重のfor文でN回Wを走査するので、時間計算量もこうなる。

## step2

コメント集から他の方のコードをみた。
自分のやった方法はフラッグの状態が遷移するのでステートマシンという分類になるようだ。

pythonにはrsplitという文字列を'@'の左右で分断してそれぞれ返すという関数があるらしく、以下のPRではそれをcppで実現している。
参考：https://github.com/Hurukawa2121/leetcode/pull/14/changes
写経させてもらった。

```cpp
class Solution {
public:
    int numUniqueEmails(vector<string>& emails) {
        vector<string> canonicalized_emails = Canonicalize(emails);
        set<string> unique_emails =
            set(canonicalized_emails.begin(), canonicalized_emails.end());
        return unique_emails.size();
    }

private:
    vector<string> Canonicalize(const vector<string>& emails) {
        auto CanonicalizeEmail = [this](const string& email) {
            size_t at_index = email.find('@');
            string local = email.substr(0, at_index);
            string domain = email.substr(at_index + 1);
            string canonicalized_local = CanonicalizeLocal(local);
            return canonicalized_local + '@' + domain;
        };
        vector<string> canonicalized_emails(emails.size());
        transform(emails.begin(), emails.end(), canonicalized_emails.begin(),
                  CanonicalizeEmail);
        return canonicalized_emails;
    }
    string CanonicalizeLocal(const string& local) {
        string local_without_plus = local.substr(0, local.find('+'));
        string local_without_plus_and_dot = local_without_plus;
        erase(local_without_plus_and_dot, '.');
        return local_without_plus_and_dot;
    }
};
```

'@'の現れるインデックスをまず記録して[0,at_index]と[at_index+1]で'@'を除いたlocalとdomainに分ける。
localから'+'と'.'を除く関数も別に作る。また、transformという関数ではイテレータを指定した範囲を一つずつ関数に入れて返り値を第三引数にコピーしてくれる。
計算量は
> 時間計算量: O(N(M+Log(N))), 空間計算量: O(N*M)

になるとのこと。全走査とsetは二分木なのでNlogNかかるんですね。
