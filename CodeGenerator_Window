#if UNITY_EDITOR
using Microsoft.CSharp;
using System.CodeDom;
using System.CodeDom.Compiler;
using System.Collections.Generic;
using System.Collections.Specialized;
using System.IO;
using System.Linq;
using System.Reflection;
using System.Text;
using UnityEditor;
using UnityEngine;

public class CodeGenerator_Window : EditorWindow
{
    private string path = @"Scripts/Auto/Axes.cs";

    private bool inputToggle = true;

    [MenuItem("Window/Code Generator")]
    private static void CallCreateWindow()
    {
        // Get existing open window or if none, make a new one:
        CodeGenerator_Window window = (CodeGenerator_Window)EditorWindow.GetWindow(typeof(CodeGenerator_Window));
        window.autoRepaintOnSceneChange = true;
        window.title = "Code Generator";
        window.Show();
    }

    private void OnInspectorUpdate()
    {
        this.Repaint();
    }

    private void OnGUI()
    {
        EditorGUILayout.BeginVertical(EditorStyles.inspectorDefaultMargins);
        EditorGUILayout.LabelField("Input", EditorStyles.boldLabel);

        this.path = EditorGUILayout.TextField(@"Path: ""Assets/"" + ", this.path);
        if (GUILayout.Button("Generate"))
        {
            try
            {
                Generate(this.path);
            }
            catch (System.Exception ex)
            {
                Debug.LogException(ex);
            }
        }

        EditorGUILayout.EndVertical();
    }

    private static void Generate(string path)
    {
        var fullPath = Path.Combine(Application.dataPath, path);

        var compileUnit = new CodeCompileUnit();

        var @namespace = new CodeNamespace();

        var name = Path.GetFileNameWithoutExtension(fullPath);
        var @type = new CodeTypeDeclaration(name);
        @type.TypeAttributes |= TypeAttributes.Sealed;

        @type.Members.Add(new CodeConstructor
        {
            Attributes = MemberAttributes.Private | MemberAttributes.Final
        });

        var axes = GetAllAxisNames();
        foreach (var axe in axes)
        {
            var @const = new CodeMemberField(
                typeof(string),
                ConvertToValidIdentifier(axe));
            @const.Attributes &= ~MemberAttributes.AccessMask;
            @const.Attributes &= ~MemberAttributes.ScopeMask;
            @const.Attributes |= MemberAttributes.Public;
            @const.Attributes |= MemberAttributes.Const;

            @const.InitExpression = new CodePrimitiveExpression(axe);
            @type.Members.Add(@const);
        }

        @namespace.Types.Add(@type);
        compileUnit.Namespaces.Add(@namespace);

        Directory.CreateDirectory(Path.GetDirectoryName(fullPath));
        using (var stream = new StreamWriter(fullPath, append:false))
        {
            var tw = new IndentedTextWriter(stream);
            var codeProvider = new CSharpCodeProvider();
            codeProvider.GenerateCodeFromCompileUnit(compileUnit, tw, new CodeGeneratorOptions());
        }

        AssetDatabase.ImportAsset(fullPath, ImportAssetOptions.ForceUpdate);
        AssetDatabase.Refresh();
    }

    private static IEnumerable<string> GetAllAxisNames()
    {
        var result = new StringCollection();

        var serializedObject = new SerializedObject(AssetDatabase.LoadAllAssetsAtPath("ProjectSettings/InputManager.asset")[0]);
        var axesProperty = serializedObject.FindProperty("m_Axes");

        axesProperty.Next(true);
        Debug.Log(axesProperty.ToString());
        axesProperty.Next(true);

        while (axesProperty.Next(false))
        {
            SerializedProperty axis = axesProperty.Copy();
            axis.Next(true);
            result.Add(axis.stringValue);
        }

        return result.AsEnumerable().Distinct();
    }

    private static string ConvertToValidIdentifier(string name)
    {
        var sb = new StringBuilder(name.Length + 1);

        if (!char.IsLetter(name[0]))
            sb.Append('_');

        var makeUpper = false;
        foreach (var ch in name)
        {
            if (char.IsLetterOrDigit(ch))
            {
                sb.Append(makeUpper
                    ? char.ToUpperInvariant(ch)
                    : ch);
                makeUpper = false;
            }
            else if (char.IsWhiteSpace(ch))
            {
                makeUpper = true;
            }
            else
            {
                sb.Append('_');
            }
        }

        return sb.ToString();
    }
}
#endif