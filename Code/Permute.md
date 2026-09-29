// Online Java Compiler (Editor)
// Write and run Java online using this editor.
import java.util.*;

class Main {
    public static void main(String[] args) {
        var str = new String("xyz");
        var strArray = str.toCharArray();
        Arrays.sort(strArray);
        var answer = new ArrayList<String>();
        //var initialString = ;

        permute2(0,strArray,answer);

        //permute(0,strArray,new String(""), answer);

        //Arrays.sort(answer);
        for(int i=0; i< answer.size(); i++){
            System.out.println(answer.get(i));
        }
        
    }

        private static void permute2(int start, char[] input, ArrayList<String> answer){
            if(start >= input.length){
                return;
            }
            
            var newString = Character.toString(input[start]);
            var newAnswer = new ArrayList<String>();

            newAnswer.add(newString);
            for(int i=0; i<answer.size(); i++){
                newAnswer.add(answer.get(i) + newString);
            }
            

            answer.addAll(newAnswer);
            permute2(start+1, input, answer);
            
        }

    private static void permute(int start, char[] input, String initialString, ArrayList<String> answer){
        //System.out.println(start);
        if(start == input.length){
             if(!initialString.isEmpty()){
                          answer.add(initialString);
   
             }
            
            return ;
        }
        permute(start + 1,input, initialString, answer);
        permute(start + 1,input, initialString+input[start], answer);
    }
}
