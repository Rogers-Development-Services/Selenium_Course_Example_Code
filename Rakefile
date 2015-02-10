require 'redcarpet'
require 'redcarpet/render_strip'
require 'erb'
require 'pygments'
require 'docverter'
require 'zip/zipfilesystem'

@template = <<HERE
<html>
  <head>
    <title>The Selenium Guidebook: How To Use Selenium, successfully</title>
    <style type="text/css">
      @font-face {
        font-family: 'Droid Sans';
        font-style: normal;
        font-weight: 400;
        src: url('droid_sans.ttf');
        -fs-pdf-font-embed: embed;
        -fs-pdf-font-encoding: Identity-H;
      }
      body {
        font-family: 'Droid Sans';
      }
      div.page_footer {
      }
      h1 {
        page-break-before: always;
      }
      pre {
        white-space: pre;
        white-space: pre-wrap;
        page-break-inside: avoid;
        orphans: 0;
        widows: 0;
        background-color: #f5f5f5;
        border: 1px solid #ccc;
        border: 1px solid rgba(0, 0, 0, 0.15);
        border-radius: 4px;
        display: block;
        padding: 9.5px;
        margin: 0 0 10px;
        font-size: 13px;
        line-height: 20px;
      }
      code {
          padding: 2px 4px;
          background-color: #f7f7f9;
      }
      img {
        width: 600px;
      }
      <%= Pygments.css %>
    </style>
  </head>
  <body>
    <%= @content %>
  </body>
</html>
HERE

class HighlightedCopyWithChapterNumbering < Redcarpet::Render::HTML

  def header(text, header_level)
    if header_level == 1
      @counter ||= 0
      @counter += 1
      @subcounter ||= 0
      @postcounter ||= 0
      if @counter == appendix_chapter
        "<h1 id=\"chapter#{@counter}\"><small>Chapter #{@counter}</small><br>#{text}</h1>\n"
      end
      if @counter > appendix_chapter
        if @subcounter <= (number_of_appendix_chapters - 1)
          @subcounter += 1
          "<h1 id=\"chapter#{appendix_chapter}_#{@subcounter}\"><small>Chapter #{appendix_chapter}.#{@subcounter}</small><br>#{text}</h1>\n"
        else
          @postcounter += 1
          "<h1 id=\"chapter#{appendix_chapter + @postcounter}\"><small>Chapter #{appendix_chapter + @postcounter}</small><br>#{text}</h1>\n"
        end
      else
        "<h1 id=\"chapter#{@counter}\"><small>Chapter #{@counter}</small><br>#{text}</h1>\n"
      end
    else
      "<h#{header_level}>#{text}</h#{header_level}>\n"
    end
  end

  def block_code(code, language)
    Pygments.highlight(code, :lexer => language)
  end

  def postprocess(document)
    document.gsub('&#39;', "'")
  end

end

class CopyWithNoFrills < Redcarpet::Render::HTML

  def block_code(code, language)
    Pygments.highlight(code, :lexer => language)
  end

  def postprocess(document)
    document.gsub('&#39;', "'")
  end

end

class TOCwithChapterNumbering < Redcarpet::Render::StripDown

  attr_reader :chapters

  def header(text, header_level)
    return unless header_level == 1
    @chapters ||= []
    @chapters << text
    ""
  end

  def postprocess(document)
    items = []
    @subcounter = 0
    @postcounter = 0
    @chapters.each_with_index do |text, chapter_number|
      if chapter_number == appendix_chapter
        items << "<ul><ol>"
        @subcounter += 1
      end
      if @subcounter == number_of_appendix_chapters
        items << "</ol></ul>"
      end
      if chapter_number > appendix_chapter
        @subcounter += 1
        if @subcounter <= number_of_appendix_chapters
          items << "<li><a href=\"#chapter#{appendix_chapter}_#{@subcounter}\">#{text}</a></li>"
        else
          @postcounter += 1
          items << "<li><a href=\"#chapter#{appendix_chapter + @postcounter}\">#{text}</a></li>"
        end
      else
        items << "<li><a href=\"#chapter#{chapter_number+1}\">#{text}</a></li>"
      end
    end
    return <<HERE
<ol>
#{items.join("\n")}
</ol>
HERE
  end

end

@renderer_toc         = Redcarpet::Markdown.new(TOCwithChapterNumbering, :fenced_code_blocks => true)
@renderer_content     = Redcarpet::Markdown.new(HighlightedCopyWithChapterNumbering, :fenced_code_blocks => true)
@renderer_no_frills   = Redcarpet::Markdown.new(CopyWithNoFrills, :fenced_code_blocks => true)

def vacuum
  tmp = ""
  tmp << yield
  tmp << "\n\n"
  tmp
end

def cover
  @renderer_no_frills.render( vacuum { File.read('content/cover.md') })
end

def acknowledgements
  @renderer_no_frills.render( vacuum { File.read('content/acknowledgements.md') })
end

def preface
  @renderer_no_frills.render( vacuum { File.read('content/preface.md') })
end

def toc
  '<h1>Table of Contents</h1>' + @renderer_toc.render(raw_content)
end

def raw_content
  tmp = ""
  appendix_chapter.times do |chapter_number|
    tmp << File.read("content/chapters/#{chapter_number + 1}.md")
    tmp << "\n\n"
  end
  Dir.glob('content/chapters/appendix/*.md').each do |appendix|
    tmp << File.read(appendix)
    tmp << "\n\n"
  end
  (number_of_chapters - appendix_chapter).times do |chapter_number|
    tmp << File.read("content/chapters/#{appendix_chapter + chapter_number + 1}.md")
  end
  tmp
end

def content
  @renderer_content.render(raw_content)
end

def raw_sample_content
  tmp = ""
  tmp << File.read('content/chapters/5.md')
  tmp << "\n\n"
  tmp << File.read('content/chapters/6.md')
  tmp << "\n\n"
  tmp
end

def sample_content
  @renderer_no_frills.render(raw_sample_content)
end

def number_of_chapters
  Dir.glob('content/chapters/*.md').count
end

def number_of_appendix_chapters
  Dir.glob('content/chapters/appendix/*.md').count
end

def appendix_chapter
  17
end

namespace :generate do
  desc 'Generate HTML'
  task :html do
    @content =  cover + preface + acknowledgements + toc + content
    html = ERB.new(@template).result(binding)
    `rm -rf output/html`
    `mkdir output/html`
    File.open('output/html/The Selenium Guidebook.html', 'w+') { |f| f.write html }
    `cp assets/* output/html`
  end

  desc 'Generate PDF'
  task :pdf do
    @content =  cover + preface + acknowledgements + toc + content
    html = ERB.new(@template).result(binding)

    File.open('output/The Selenium Guidebook.pdf', 'w+') do |f|
      f.write(Docverter::Conversion.run do |c|
        c.from    = 'html'
        c.to      = 'pdf'
        c.content = html
        Dir.glob('assets/*') do |asset|
          c.add_other_file asset
        end
      end)
    end
  end

#  desc 'Generate PDF Sample Chapter'
#  task :preview do
#    @content =  cover + toc + sample_content
#    html = ERB.new(@template).result(binding)
#
#    File.open('output/The Selenium Guidebook Sample.pdf', 'w+') do |f|
#      f.write(Docverter::Conversion.run do |c|
#        c.from    = 'html'
#        c.to      = 'pdf'
#        c.content = html
#        Dir.glob('assets/*') do |asset|
#          c.add_other_file asset
#        end
#      end)
#    end
#  end

  desc 'Generate MOBI'
  task :mobi do
    File.open('output/The Selenium Guidebook.mobi', 'w+') do |file|
      mobi = Docverter::Conversion.run do |c|
        c.from              = 'markdown'
        c.to                = 'mobi'
        c.content           = raw_content
        c.epub_metadata     = 'metadata.xml'
  #      c.epub_cover_image  = 'bookcover.png'
        c.epub_stylesheet   = 'epub.css'
        c.add_other_file    'assets/epub.css'
        c.add_other_file    'assets/metadata.xml'
  #      c.add_other_file    'assets/bookcover.png'
      end

      file.write mobi
    end
  end

  desc 'Generate EPUB'
  task :epub do
    File.open('output/The Selenium Guidebook.epub', 'w+') do |file|
      epub = Docverter::Conversion.run do |c|
        c.from              = 'markdown'
        c.to                = 'epub'
        c.content           = raw_content
        c.epub_metadata     = 'metadata.xml'
  #      c.epub_cover_image  = 'bookcover.png'
        c.epub_stylesheet   = 'epub.css'
        c.add_other_file    'assets/epub.css'
        c.add_other_file    'assets/metadata.xml'
  #      c.add_other_file    'assets/bookcover.png'
      end

      file.write epub
    end
  end

#  desc 'Generate everything'
#  task :all => ['generate:html', 'generate:pdf', 'generate:preview', 'generate:mobi', 'generate:epub']
end

def make_zip_file(name, files)
  Zip::ZipFile.open(name, Zip::ZipFile::CREATE) do |zf|
    files.each do |file|
      if file =~ /\/$/
        zf.dir.mkdir(file)
        next
      end

      zf.file.open(file, "w") { |f| f.write File.read(file) }
    end
  end
end

def pull_in_code_examples
  `rm output/code_examples.zip`
  `zip -r output/code_examples.zip code_examples/`
end

namespace :package do

  task :common do
    `cp *.txt output`
  end

  desc 'Package up zip of book with code for individuals'
  task :individual => :common do
    pull_in_code_examples
    Dir.chdir('output') do
      `rm selenium_guidebook_individual.zip`
      make_zip_file('selenium_guidebook_individual.zip', [
          'The Selenium Guidebook.pdf',
          'The Selenium Guidebook.mobi',
          'The Selenium Guidebook.epub',
          'html/',
          'html/The Selenium Guidebook.html',
          'html/droid_sans.ttf',
          'html/bookcover.png',
          'copyright.txt',
          'individual_license.txt',
          'code_examples.zip'
      ])
    end
  end

  namespace :team do

    desc 'Package up zip of book with code for small teams'
    task :small => :common do
      Dir.chdir('output') do
        `rm selenium_guidebook_small_team.zip`
        make_zip_file('selenium_guidebook_small_team.zip', [
            'The Selenium Guidebook.pdf',
            'The Selenium Guidebook.mobi',
            'The Selenium Guidebook.epub',
            'html/',
            'html/The Selenium Guidebook.html',
            'html/droid_sans.ttf',
            'html/bookcover.png',
            'copyright.txt',
            'small_team_license.txt',
            'code_examples.zip'
        ])
      end
    end

    desc 'Package up zip of book with code for big teams'
    task :big => :common do
      Dir.chdir('output') do
        `rm selenium_guidebook_big_team.zip`
        make_zip_file('selenium_guidebook_big_team.zip', [
            'The Selenium Guidebook.pdf',
            'The Selenium Guidebook.mobi',
            'The Selenium Guidebook.epub',
            'html/',
            'html/The Selenium Guidebook.html',
            'html/droid_sans.ttf',
            'html/bookcover.png',
            'copyright.txt',
            'big_team_license.txt',
            'code_examples.zip'
        ])
      end
    end
  end

#  desc 'Package everything'
#  task :all => ['package:book', 'package:book_with_code', 'package:team']
end

#desc "z'Big Red Button"
#task :z_big_red_button => ['generate:all', 'package:all']
